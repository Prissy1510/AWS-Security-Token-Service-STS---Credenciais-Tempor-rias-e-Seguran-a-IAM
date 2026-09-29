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

### 📜 Script Python de Automação (credenciais_temporarias.py)

Para automatizar a solicitação das credenciais via SDK Boto3, foi desenvolvido o script `credenciais_temporarias.py`. O código consome o serviço AWS STS através do método `assume_role` e trata exceções como limites de tempo de sessão:

```python
import boto3
import sys
import argparse
from botocore.exceptions import ClientError


def gerar_credenciais_temporarias(role_arn, session_name, duration):
    # Inicializa o cliente STS
    sts_client = boto3.client('sts')

    try:
        # Solicita as credenciais temporárias
        response = sts_client.assume_role(
            RoleArn=role_arn,
            RoleSessionName=session_name,
            DurationSeconds=duration
        )

        # Exibe as credenciais temporárias
        credentials = response['Credentials']
        print(f"AWS Access Key ID: {credentials['AccessKeyId']}")
        print(f"AWS Secret Access Key: {credentials['SecretAccessKey']}")
        print(f"AWS Session Token: {credentials['SessionToken']}")

    except ClientError as e:
        error_code = e.response['Error']['Code']
        error_msg = e.response['Error']['Message']

        if error_code == 'ValidationError' and 'DurationSeconds exceeds the MaxSessionDuration' in error_msg:
            print("[ERRO] A duração solicitada excede o limite de tempo máximo permitido para esta role.")
            print("Por favor, solicite ao administrador da role para aumentar o limite de tempo.")
            sys.exit(1)
        else:
            print(f"[ERRO] Falha ao obter credenciais: {e}")
            sys.exit(1)


def main():
    parser = argparse.ArgumentParser(description="Gerar credenciais temporárias do AWS STS")
    parser.add_argument('--role-arn', required=True, help='ARN da role a ser assumida')
    parser.add_argument('--session-name', required=True, help='Nome único para a sessão')
    parser.add_argument('--duration', type=int, default=3600, help='Duração em segundos (padrão: 3600 = 1 hora)')

    args = parser.parse_args()

    gerar_credenciais_temporarias(args.role_arn, args.session_name, args.duration)


if __name__ == "__main__":
    main()
```
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
---

## 🧠 Aprendizados Adquiridos

A realização deste laboratório prático proporcionou a consolidação de conceitos fundamentais de segurança, gerenciamento de identidade e automação na nuvem Amazon Web Services (AWS). Os principais conhecimentos e competências técnicas desenvolvidos foram:

* **Gestão Dinâmica de Identidades com AWS STS**: Compreensão profunda sobre a importância da substituição de credenciais estáticas de longa duração por credenciais temporárias (`AccessKey`, `SecretKey` e `SessionToken`) com tempo de expiração definido, reduzindo drasticamente a superfície de ataque.

* **Delegação de Permissões e Assunção de Papéis (Role Assumption)**: Domínio do fluxo de assunção de IAM Roles via CLI e SDK Python, entendendo o papel das Políticas de Confiança (*Trust Policies*) e Políticas de Acesso (*Identity-based Policies*).

* **Aplicação Prática do Princípio do Menor Privilégio (PoLP)**: Validação do controle de acesso granular ao conceder permissão total ao Amazon S3 e confirmar o bloqueio ostensivo de ações não autorizadas em outros serviços, como o AWS Lambda (`AccessDeniedException`).

* **Automação com Python e SDK Boto3**: Capacidade de criar e executar scripts automatizados para interagir com a API do AWS STS, tratar exceções de limites de tempo de sessão e manipular payloads de resposta.

* **Gerenciamento de Sessões na AWS CLI**: Domínio na configuração manual de tokens de sessão temporários no arquivo `~/.aws/credentials` e no uso de perfis temporários para execução de comandos CLI.

* **Auditoria, Rastreabilidade e Validação de Acesso**: Utilização do comando `aws sts get-caller-identity` para auditar em tempo real a entidade ativa no ambiente, permitindo comprovar a transição entre o usuário IAM inicial e a Role assumida.
---