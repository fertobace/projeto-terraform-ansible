# 🚀 Projeto Terraform + Ansible — Deploy Automático com Python e Django

Este projeto demonstra como **criar e configurar automaticamente uma instância EC2 na AWS** usando **Terraform** (para provisionamento da infraestrutura) e **Ansible** (para automação da configuração do servidor e deploy de um ambiente Python/Django).

---

## 🧩 Estrutura do Projeto

```
.
├── terraform/
│   ├── main.tf
│   └── nome-da-chave.pem
│
├── ansible/
│   ├── hosts.ini
│   └── playbook.yml
│
└── README.md
```

---

## ☁️ Parte 1 — Terraform (Infraestrutura)

O Terraform é responsável por criar a **instância EC2 Ubuntu 24.04** na AWS.

### 📄 Arquivo `main.tf`

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.92"
    }
  }

  required_version = ">= 1.2"
}

provider "aws" {
  region = "us-east-2"
}

data "aws_ami" "ubuntu" {
  most_recent = true

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd-gp3/ubuntu-noble-24.04-amd64-server-*"]
  }

  owners = ["099720109477"] # Canonical
}

resource "aws_instance" "app_server" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t3.small"
  key_name      = "nome-da-chave"
  tags = {
    Name = "terraform ansible python"
  }
}
```

### ▶️ Comandos principais

1. Inicializar o Terraform:
   ```bash
   terraform init
   ```

2. Gerar o plano de execução:
   ```bash
   terraform plan
   ```

3. Criar a infraestrutura:
   ```bash
   terraform apply
   ```

Após a execução, será criada uma instância EC2 rodando Ubuntu 24.04 LTS.  
Use o IP público dessa instância no inventário do Ansible.

---

## ⚙️ Parte 2 — Ansible (Configuração)

O Ansible automatiza a configuração da instância, instalando o Python, criando o ambiente virtual, instalando Django e configurando um projeto pronto para desenvolvimento.

### 📄 Arquivo `playbook.yml`

```yaml
---
- hosts: terraform-ansible
  become: true
  tasks:
    - name: Instalar Python3 e pip
      apt:
        pkg:
          - python3
          - python3-venv
          - python3-pip
        update_cache: yes

    - name: Criar diretório do projeto
      file:
        path: /home/ubuntu/projeto
        state: directory

    - name: Criar ambiente virtual com venv
      command: python3 -m venv /home/ubuntu/projeto/venv

    - name: Instalar dependências dentro do venv
      pip:
        virtualenv: /home/ubuntu/projeto/venv
        name:
          - django
          - djangorestframework

    - name: Iniciando o Projeto
      shell: ". /home/ubuntu/projeto/venv/bin/activate; django-admin startproject setup /home/ubuntu/projeto/"

    - name: Alterando o host do settings
      lineinfile: 
        path: /home/ubuntu/projeto/setup/settings.py
        regexp: 'ALLOWED_HOSTS'
        line: 'ALLOWED_HOSTS = ["*"]'
        backrefs: yes 
```

### 📄 Arquivo `hosts.ini`

```ini
[terraform-ansible]
SEU_IP_PUBLICO_DA_EC2
```

---

### ▶️ Executando o Ansible

1. Acesse a pasta do Ansible:
   ```bash
   cd ansible
   ```

2. Execute o playbook:
   ```bash
   ansible-playbook playbook.yml -u ubuntu --private-key caminho-da-chave -i hosts.ini
   ```

O Ansible vai:
- Instalar Python e pip;
- Criar o diretório `/home/ubuntu/projeto`;
- Criar o ambiente virtual `/home/ubuntu/projeto/venv`;
- Instalar **Django** e **Django REST Framework**;
- Criar automaticamente um projeto Django chamado **setup**;
- Configurar o `ALLOWED_HOSTS` para aceitar todas as conexões.

---

## 🧠 Conceitos Envolvidos

| Ferramenta | Função |
|-------------|--------|
| **Terraform** | Criação da infraestrutura na AWS (IaC) |
| **Ansible** | Automação da configuração do ambiente |
| **Python / venv** | Criação de ambiente isolado de desenvolvimento |
| **Django + DRF** | Frameworks para construção de APIs e aplicações web |

---

## ✅ Resultado Final

Ao final da execução:
- Uma instância EC2 Ubuntu é criada automaticamente via Terraform;
- O Ansible acessa essa instância e prepara um ambiente completo com Python, virtualenv e Django;
- Um projeto Django inicial (`setup`) estará disponível em `/home/ubuntu/projeto/`.

---

## 🔒 Requisitos

- Conta AWS configurada (`aws configure`);
- Chave PEM (`nome-da-chave.pem`);
- Terraform instalado;
- Ansible instalado;
- Python 3 instalado na máquina local.

---

## 📜 Licença

Este projeto é de uso educacional e livre para fins de estudo e aprendizado.
