
---

# Documentação: Portal de Solicitação de Recursos em Nuvem

## Visão Geral
Este documento fornece um guia passo a passo para desenvolver um portal web que permite a solicitação e o provisionamento de recursos em nuvens como AWS e Azure. A stack tecnológica utilizada inclui Python, Flask/Django e Terraform.

## 1. Definição dos Requisitos
### Recursos
- Instâncias EC2 (AWS)
- Máquinas Virtuais (Azure)
- Bancos de Dados
- Ambientes (DEV, HML, PROD)

### Funcionalidades
- Interface de usuário para solicitar recursos
- Autenticação e autorização
- Provisionamento automatizado de infraestrutura

## 2. Configuração do Ambiente
### Instalando Dependências
1. **Python**:
   ```bash
   sudo apt-get update
   sudo apt-get install python3 python3-pip
   ```

2. **Virtual Environment**:
   ```bash
   pip install virtualenv
   virtualenv venv
   source venv/bin/activate  # Em Linux/Mac
   venv\Scripts\activate  # Em Windows
   ```

3. **Flask/Django**:
   ```bash
   pip install flask  # ou pip install django
   ```

4. **Terraform**:
   - Baixe e instale [Terraform](https://www.terraform.io/downloads.html).

## 3. Desenvolvimento do Portal
### Estrutura do Projeto
```
my-portal/
├── app/
│   ├── __init__.py
│   ├── routes.py
│   ├── templates/
│   ├── static/
│   └── terraform/
│       ├── main.tf
│       ├── variables.tf
│       └── outputs.tf
├── venv/
├── requirements.txt
└── run.py
```

### Backend (Flask Exemplo)
**app/routes.py**:
```python
from flask import Flask, request, render_template
import os

app = Flask(__name__)

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/deploy', methods=['POST'])
def deploy():
    resource_type = request.form['resource_type']
    os.system(f'terraform -chdir=terraform apply -var="type={resource_type}" -auto-approve')
    return 'Deploy iniciado!', 200

if __name__ == '__main__':
    app.run(debug=True)
```

**app/templates/index.html**:
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Portal de Recursos</title>
</head>
<body>
    <h1>Solicitação de Recursos</h1>
    <form action="/deploy" method="POST">
        <label for="resource_type">Tipo de Recurso:</label>
        <input type="text" id="resource_type" name="resource_type">
        <button type="submit">Solicitar</button>
    </form>
</body>
</html>
```

## 4. Automação com Terraform
### Exemplo de Arquivo Terraform (main.tf)
```hcl
provider "aws" {
  region = "us-west-2"
}

resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  tags = {
    Name = "ExampleInstance"
  }
}
```

### Comandos Terraform
1. Inicialização:
   ```bash
   terraform -chdir=app/terraform init
   ```

2. Planejamento:
   ```bash
   terraform -chdir=app/terraform plan
   ```

3. Aplicação:
   ```bash
   terraform -chdir=app/terraform apply -auto-approve
   ```

## 5. Autenticação e Autorização
### Flask-Login (Exemplo)
```bash
pip install flask-login
```

**app/routes.py**:
```python
from flask_login import LoginManager, UserMixin, login_user, login_required, logout_user

login_manager = LoginManager()
login_manager.init_app(app)

class User(UserMixin):
    # Implementação da classe User

@login_manager.user_loader
def load_user(user_id):
    # Implementação para carregar usuário

@app.route('/login', methods=['GET', 'POST'])
def login():
    # Implementação da rota de login

@app.route('/logout')
@login_required
def logout():
    logout_user()
    return 'Desconectado!'
```

## 6. Implementação de CI/CD
### Exemplo GitHub Actions
**.github/workflows/deploy.yml**:
```yaml
name: Deploy Terraform

on:
  push:
    branches:
      - main

jobs:
  terraform:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Setup Terraform
      uses: hashicorp/setup-terraform@v1

    - name: Terraform Init
      run: terraform -chdir=app/terraform init

    - name: Terraform Plan
      run: terraform -chdir=app/terraform plan

    - name: Terraform Apply
      run: terraform -chdir=app/terraform apply -auto-approve
```

## 7. Monitoramento e Suporte
- Utilize ferramentas como **AWS CloudWatch** e **Azure Monitor** para monitorar os recursos provisionados.
- Documente detalhadamente as funcionalidades do portal para auxiliar os usuários.

---
