# 🚀 AWS Security Token Service (STS) - Credenciais Temporárias e Segurança IAM

Este laboratório demonstra a implementação e utilização de credenciais temporárias na AWS via **AWS Security Token Service (STS)**. O projeto abrange a criação e assunção de Roles IAM com permissões restritas, execução de scripts Python no CloudShell para geração de tokens temporários, e validação prática de acessos permitidos e negados conforme os princípios de menor privilégio.

---

## 🏗️ Arquitetura do Laboratório

O diagrama abaixo ilustra o fluxo de autenticação e autorização via AWS STS implementado neste laboratório:

![Arquitetura do Laboratório - AWS STS](imagens/Captura%20de%20Tela%20(2106).png)

---

## 🛠️ Passo a Passo da Implementação

### 1. Seleção de Role no Console AWS (Acesso Temporário)

Alternância para a Role IAM temporária `Temporario-PriscillaCorrea` através do console web da AWS para navegação segura com permissões restritas.

![Seleção de Role no Console AWS](imagens/Captura%20de%20Tela%20(2202).png)

---

### 2. Verificação de Identidade Inicial no CloudShell

Abertura do terminal AWS CloudShell e execução do comando `aws sts get-caller-identity` para identificar o usuário IAM ativo antes da assunção de papéis:

```bash
aws sts get-caller-identity
```

![Verificação de Identidade Inicial no CloudShell](imagens/Captura%20de%20Tela%20(2203).png)

---

### 3. Geração de Credenciais Temporárias via Script Python

Execução do script `credenciais_temporarias.py` no CloudShell solicitando a assunção da role `Role-PriscillaCorrea`. Após validar a limitação de tempo para 3600 segundos (1 hora), foram geradas as chaves `AWS Access Key ID`, `AWS Secret Access Key` e o `AWS Session Token`:

```bash
python3 credenciais_temporarias.py --role-arn arn:aws:iam::231095867761:role/Role-PriscillaCorrea --session-name AcessoTemporario --duration 3600
```

![Geração de Credenciais Temporárias via Script Python](imagens/Captura%20de%20Tela%20(2204).png)

---

### 4. Configuração das Credenciais na AWS CLI

Inclusão e exportação das credenciais temporárias geradas (Access Key, Secret Key e Session Token) nas configurações da AWS CLI através do comando `aws configure`:

```bash
aws configure
```

![Configuração das Credenciais na AWS CLI](imagens/Captura%20de%20Tela%20(2205).png)

---

### 5. Validação de Acesso Permitido ao Amazon S3

Execução do comando `aws s3 ls` utilizando as credenciais temporárias ativas. Como a Role possui a política `AmazonS3FullAccess` atribuída, a listagem de buckets foi realizada com sucesso:

```bash
aws s3 ls
```

![Validação de Acesso Permitido ao Amazon S3 - Parte 1](imagens/Captura%20de%20Tela%20(2206).png)
![Validação de Acesso Permitido ao Amazon S3 - Parte 2](imagens/Captura%20de%20Tela%20(2207).png)

---

### 6. Alternando para a Role Temporária e Comprovação de Identidade (Avaliação)

Execução do comando `aws sts get-caller-identity` para comprovar que a identidade atual é a role assumida (`assumed-role/Role-PriscillaCorrea/AcessoTemporario`):

```bash
aws sts get-caller-identity
```

![Comprovação de Identidade da Role Assumida](imagens/Captura%20de%20Tela%20(2208).png)

---

### 7. Validação de Acesso Negado ao AWS Lambda

Testando o limite de permissões da role temporária executando `aws lambda list-functions`. A requisição foi bloqueada com erro de `AccessDeniedException`, comprovando a eficácia das políticas restritivas do IAM:

```bash
aws lambda list-functions
```

![Validação de Acesso Negado ao AWS Lambda](imagens/Captura%20de%20Tela%20(2209).png)

---

### 8. Validação da Política de Confiança e Encerramento

Confirmação do comportamento das políticas de confiança (Trust Policy) e revogação/expiração automática do token temporário após o encerramento do ciclo de vida:

![Validação Final e Limpeza](imagens/Captura%20de%20Tela%20(2210).png)

---

## 🎯 Resultado da Avaliação

* **Comprovação de Identidade Assumida (`aws sts get-caller-identity`)**:
```json
{
    "UserId": "AROATLTTDDFY5ZMQP502K:AcessoTemporario",
    "Account": "231095867761",
    "Arn": "arn:aws:sts::231095867761:assumed-role/Role-PriscillaCorrea/AcessoTemporario"
}
```