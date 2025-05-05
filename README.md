# desafio-aws-vpc-ec2-web-server-run-as-cloud
Aprenda a criar uma Virtual Private Cloud (VPC) personalizada, configurar sub-redes e executar um servidor web simples (Apache) em uma instância EC2, tudo dentro do Free Tier da AWS.  Este é um laboratório prático do Grupo de Estudos Run as Cloud.

---

## ⚠️ **Atenção antes de começar**

Este laboratório utiliza **recursos reais da AWS**. Embora todos estejam **dentro do Free Tier**, é essencial:

- Monitorar o uso no **AWS Billing Dashboard** e no **Cost Explorer**
- **Encerrar recursos** (como instâncias EC2) ao final
- **Evitar publicar dados sensíveis** em prints ou repositórios

📌 Evite expor:
- ID da conta AWS
- URL de login do IAM
- Senhas ou chaves
- Prints com dados pessoais

---

## 📚 **Objetivo**

- Criar uma VPC com sub-rede pública
- Associar um gateway de internet
- Configurar um grupo de segurança
- Executar uma instância EC2 (Amazon Linux) com Apache Web Server
- Acessar a página de boas-vindas via navegador

---

🚫 **Recursos eliminados para manter o laboratório 100% Free Tier**:

- ❌ NAT Gateway (gera custo)
- ❌ Sub-redes privadas
- ✅ Usaremos apenas uma **sub-rede pública** com acesso à internet

---

## 🏗️ **Etapas do laboratório**

1. Criar a VPC (bloco CIDR 10.0.0.0/16)
2. Criar uma sub-rede pública (ex: 10.0.0.0/24)
3. Criar e associar um Internet Gateway
4. Criar uma tabela de rotas para acesso externo
5. Criar um grupo de segurança permitindo acesso HTTP (porta 80)
6. Lançar uma instância EC2 (t2.micro) com Apache via script de usuário
7. Acessar a página de boas-vindas "Servidor Web Run as Cloud"

---

## 🟩 Etapa 1 – Criar a VPC
Passos:
- No Console AWS, vá para VPC > Sua VPCs.

- Clique em Criar VPC.

Preencha:
 - Selecione a opção `VPC Only`.

 - Nome: `lab-vpc`

 - Bloco CIDR IPv4: `10.0.0.0/16`

Clique em Criar VPC.

📸 
Tela com os dados preenchidos para criação da VPC (10.0.0.0/16).

![image (54)](https://github.com/user-attachments/assets/fa535322-4201-4780-9e2a-18666a1ced8d)

Resumo da VPC criada com nome lab-vpc.

![image (55) (1)](https://github.com/user-attachments/assets/d4bfbc35-4518-47e1-a18f-cb7dea8b4686)

---
## 🟨 Etapa 2 – Criar Sub-rede Pública
Passos:
Vá para Sub-redes > Criar sub-rede.

Preencha:
- VPC: `lab-vpc`

- Nome da sub-rede: `lab-subnet-public`

- Zona de disponibilidade: `us-east-1a`

- Bloco CIDR IPv4 da sub-rede: `10.0.0.0/24`

Clique em Criar sub-rede.

📸 
Tela de criação da sub-rede com os dados acima.

![image (56)](https://github.com/user-attachments/assets/7fdc7442-7b02-42f7-9f66-068192ba82a7)


Sub-rede criada listada no console.

![image (57)](https://github.com/user-attachments/assets/619d19f5-c4d7-46da-b1af-e4a63de48bdb)

## 🟧 Etapa 3 – Criar e Associar o Internet Gateway
Passos:
Vá para Internet Gateways > Criar internet gateway.

Preencha:
 - Nome: `lab-igw`

Clique em Criar internet gateway.

Após criado, selecione o IGW > Ações > Associar à VPC > selecione `lab-vpc`

📸
Tela de criação do IGW com nome lab-igw.

![image (58)](https://github.com/user-attachments/assets/67a73585-4354-4c96-b1c2-0a83397de4ee)


Tela de associação do IGW à VPC lab-vpc.

![image (59)](https://github.com/user-attachments/assets/f4e34a95-5584-4f58-a166-01e1ce294b54)

---

## 🟦 Etapa 4 – Criar Tabela de Rotas e Associar Sub-rede
Passos:
Vá para Tabelas de rotas > Criar tabela de rotas.

Preencha:
 - Nome: `lab-public-rt`

 - VPC: `lab-vpc`

Clique em Criar.

Selecione a tabela criada > Ações > Editar rotas > Adicionar rota:

 - Destino: `0.0.0.0/0`

 - Target: selecione o IGW (`lab-igw`)

Salvar alterações.

Vá em Sub-redes associadas > Editar associações > selecione `lab-subnet-public`

📸 
Criação da tabela com nome lab-public-rt.

![image (60)](https://github.com/user-attachments/assets/c1ed9842-0bef-4b4a-af17-976b864ba0cf)


Adição da rota 0.0.0.0/0 com destino ao IGW.

![image (61)](https://github.com/user-attachments/assets/45e10811-b0e7-4bfa-8fe2-ff8b37dade45)

Associação da sub-rede pública.

![image (62)](https://github.com/user-attachments/assets/715f0ff5-d0a7-4d2a-a831-41e164dcbec9)

---

## 🛡️ Etapa 5 – Criar Grupo de Segurança
Passos:
Vá para Grupos de segurança > Criar grupo de segurança.

Preencha:
 - Nome: `Web Security Group` 

 - VPC: `lab-vpc`

Em Regras de entrada, adicione:

 - Tipo: `HTTP`

 - Porta: `80`

 - Origem: `0.0.0.0/0`

Clique em Criar grupo de segurança.

📸 
Tela de criação do grupo com nome e regra HTTP porta 80.

![image (64)](https://github.com/user-attachments/assets/5e3af001-2c37-4ed5-8c78-462f100ac05f)


Esse grupo permitirá que qualquer usuário na internet acesse a porta 80 (web) da sua instância EC2, o que é exatamente o que você quer para um servidor Apache simples nesse laboratório.

---

## 🖥️ Etapa 6 – Executar Instância EC2
Passos:
Vá para EC2 > Instâncias > Executar instância.

Preencha:
 - Nome da instância: `Web Server Run as Cloud`

 - AMI: `Amazon Linux 2023`

 - Tipo: `t2.micro`

 - Sub-rede: `lab-subnet-public`

 - Atribuir IP público: ✅ Habilitado

 - Grupo de segurança: selecione `Web Security Group`

Role até "Dados do usuário" e insira o script:

```
#!/bin/bash
dnf update -y
dnf install -y httpd
systemctl start httpd
systemctl enable httpd
echo "Servidor Web Run as Cloud" > /var/www/html/index.html
```
Clique em Executar instância.

📸 
Tela de configuração com os dados da instância.

![image (65)](https://github.com/user-attachments/assets/2a4b89db-f6c4-40b0-ae12-aa402bdec161)


Campo “Dados do usuário” com o script preenchido.

![image (67)](https://github.com/user-attachments/assets/7546987d-d559-459b-b725-482b1166aab8)

Para esta atividade, como o foco é lançar uma instância EC2 que execute um servidor web acessível via HTTP (porta 80) e não exige conexão SSH para fins de teste, você pode escolher a opção:

✅ Proceed without key pair

![image (68)](https://github.com/user-attachments/assets/b0ba195f-b75c-4b89-943e-99a80fcdbf17)

---

🌐 Etapa 7 – Verificar Web Server
Passos:
Vá até a instância EC2 e aguarde até o status: 2/2 checks passed.

 - Copie o IP público da instância.

 -  No navegador, acesse: `http://[IP_PÚBLICO]`

📸 
EC2 com status "2/2 checks passed".

IP público copiado.

![image (69)](https://github.com/user-attachments/assets/72a6b20f-92f6-4e6b-be2a-70027b1f77b0)

Página no navegador exibindo “Servidor Web Run as Cloud”.

❗ Se a página não abrir:

- Verifique se a instância está em execução e com o status "2/2 checks passed"
- Confirme que o grupo de segurança está permitindo tráfego na porta 80
- Tente acessar usando `http://<IP_PÚBLICO>` (sem "https://")

