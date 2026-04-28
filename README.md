# 🌐 AWS Static Website Infrastructure

> Infraestrutura serverless provisionada como código (IaC) usando AWS CloudFormation.  
> Demonstra integração de S3, Route 53, CloudWatch e IAM com princípios de least privilege.

---

## 📐 Arquitectura

```
                        ┌─────────────────────────────────────────┐
                        │              AWS Cloud                   │
                        │                                          │
  User Browser          │   Route 53          S3 Bucket           │
  ─────────────  ──────►│   (DNS)      ──────► (Static Hosting)   │
                        │                          │               │
                        │                     CloudWatch           │
                        │                     (Monitoring)         │
                        │                          │               │
                        │                      SNS Topic           │
                        │                     (Alerts → Email)     │
                        └─────────────────────────────────────────┘
```

**Serviços utilizados:**

| Serviço | Função |
|---|---|
| **S3** | Armazenamento e hosting dos ficheiros estáticos |
| **Route 53** | DNS e registo do domínio |
| **CloudWatch** | Monitorização de erros 4xx e métricas |
| **SNS** | Notificações por email via alarmes |
| **IAM** | Role com permissões mínimas (least privilege) |
| **CloudFormation** | Provisionamento de toda a infra como código |

---

## 🗂️ Estrutura do Repositório

```
aws-static-site/
├── template.yaml          # Template CloudFormation (infra completa)
├── website/
│   ├── index.html         # Página principal
│   └── error.html         # Página de erro 404
├── scripts/
│   └── deploy.sh          # Script de deploy automatizado
└── README.md
```

---

## ⚙️ Pré-requisitos

- Conta AWS com permissões de CloudFormation, S3, Route 53, IAM
- AWS CLI v2 configurado (`aws configure`)
- Domínio registado no Route 53
- MFA activado na conta (boa prática de segurança)

---

## 🚀 Deploy — Passo a Passo

### 1. Clonar o repositório

```bash
git clone https://github.com/[teu-username]/aws-static-site.git
cd aws-static-site
```

### 2. Fazer deploy do stack CloudFormation

```bash
aws cloudformation deploy \
  --template-file template.yaml \
  --stack-name static-website \
  --parameter-overrides \
      DomainName=meusite.com \
      AlarmEmail=teu@email.com \
  --capabilities CAPABILITY_NAMED_IAM \
  --region eu-west-1
```

### 3. Fazer upload dos ficheiros do site

```bash
aws s3 sync ./website/ s3://$(aws cloudformation describe-stacks \
  --stack-name static-website \
  --query "Stacks[0].Outputs[?OutputKey=='BucketName'].OutputValue" \
  --output text)
```

### 4. Confirmar o email do SNS

Após o deploy, verifica o teu email e confirma a subscrição do SNS para receber alertas do CloudWatch.

### 5. Verificar o deploy

```bash
# Ver outputs do stack (URL do site, bucket name, etc.)
aws cloudformation describe-stacks \
  --stack-name static-website \
  --query "Stacks[0].Outputs"
```

---

## 🔒 Decisões de Segurança

**Least Privilege IAM**  
A IAM Role criada só tem permissões para as acções estritamente necessárias no bucket (`s3:PutObject`, `s3:GetObject`, `s3:DeleteObject`, `s3:ListBucket`). Não tem acesso a outros serviços ou buckets.

**MFA na conta root**  
A conta AWS usa MFA activado no utilizador root e no utilizador IAM Admin, seguindo as melhores práticas AWS.

**Bucket Policy explícita**  
O acesso público ao bucket é concedido apenas através de uma Bucket Policy explícita com `s3:GetObject`, limitada aos objectos do bucket do projecto.

---

## 📊 Monitorização

O stack cria automaticamente um alarme CloudWatch que:

- Monitoriza erros **4xx** no bucket S3
- Avalia a cada **5 minutos**
- Dispara um alerta por **email via SNS** se houver mais de **10 erros** num período

Para ver as métricas na consola:
```
CloudWatch → Metrics → S3 → BucketName, FilterId
```

---

## 🧹 Limpeza (Evitar Custos)

Para apagar todos os recursos criados:

```bash
# 1. Esvaziar o bucket primeiro (obrigatório antes de apagar o stack)
aws s3 rm s3://[nome-do-bucket] --recursive

# 2. Apagar o stack completo
aws cloudformation delete-stack --stack-name static-website

# 3. Verificar que foi apagado
aws cloudformation wait stack-delete-complete --stack-name static-website
```

---

## 💰 Estimativa de Custos

| Serviço | Custo Estimado |
|---|---|
| S3 (< 1 GB, < 10k requests/mês) | ~$0.00 (Free Tier) |
| Route 53 Hosted Zone | ~$0.50/mês |
| CloudWatch (métricas básicas) | $0.00 (Free Tier) |
| SNS (< 1000 emails/mês) | $0.00 (Free Tier) |
| **Total estimado** | **~$0.50/mês** |

---

## 📚 Conceitos Demonstrados

- **Infrastructure as Code (IaC)** — toda a infra definida em CloudFormation YAML, versionada em Git
- **Least Privilege** — IAM Role com permissões mínimas necessárias
- **Observability** — CloudWatch Alarm com notificação automática
- **DNS Management** — Route 53 Hosted Zone e A Record com Alias para S3
- **Idempotência** — o template pode ser re-aplicado sem efeitos secundários

---

## 🔗 Recursos de Referência

- [AWS S3 Static Website Hosting](https://docs.aws.amazon.com/AmazonS3/latest/userguide/WebsiteHosting.html)
- [CloudFormation User Guide](https://docs.aws.amazon.com/cloudformation/)
- [Route 53 Developer Guide](https://docs.aws.amazon.com/route53/)
- [CloudWatch Alarms](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/AlarmThatSendsEmail.html)

---

*Projecto desenvolvido durante preparação para AWS Solutions Architect Associate (SAA-C03)*
