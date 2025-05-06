# desafio-aws-vpc-ec2-web-server-run-as-cloud
Aprenda a criar uma Virtual Private Cloud (VPC) personalizada, configurar sub-redes e executar um servidor web simples (Apache) em uma instância EC2, tudo dentro do Free Tier da AWS.  Este é um laboratório prático do Grupo de Estudos Run as Cloud.

## 🌐 O que é uma VPC?
A Amazon VPC (Virtual Private Cloud) permite provisionar uma rede virtual isolada dentro da nuvem da AWS, onde você pode definir seus próprios intervalos de IP, criar sub-redes, configurar tabelas de rotas e gateways de internet. É como ter seu próprio data center na nuvem — mas com toda a flexibilidade e escalabilidade da AWS.

Dentro de uma VPC, você pode controlar como suas instâncias EC2 se comunicam entre si, com outros serviços da AWS e com a internet. Você também define regras de segurança personalizadas, tornando sua infraestrutura mais segura e sob controle total.

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

![image (72)](https://github.com/user-attachments/assets/2f6ac27f-e0d7-4005-8948-fd2d72998d68)


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

![image (73)](https://github.com/user-attachments/assets/a72701ad-f475-4ad2-9d3e-196727621291)



Sub-rede criada listada no console.

![image (74)](https://github.com/user-attachments/assets/65e53ac4-e947-43fd-80bd-e03e7d8e5949)


## 🟧 Etapa 3 – Criar e Associar o Internet Gateway
Passos:
Vá para Internet Gateways > Criar internet gateway.

Preencha:
 - Nome: `lab-igw`

Clique em Criar internet gateway.

Após criado, selecione o IGW > Ações > Associar à VPC > selecione `lab-vpc`

📸
Tela de criação do IGW com nome lab-igw.

![image (75)](https://github.com/user-attachments/assets/6c5de181-d0b8-4696-bf0d-26da26e07679)



Tela de associação do IGW à VPC lab-vpc.

![image (76)](https://github.com/user-attachments/assets/1975e70d-a844-47ed-b60e-2da96605e402)

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

![image (77)](https://github.com/user-attachments/assets/2c1b64c3-c66b-4cc5-8e62-c22a9d059295)



Adição da rota 0.0.0.0/0 com destino ao IGW.

![image (78)](https://github.com/user-attachments/assets/d2deda26-4cfe-4c99-b2b3-68d409f085c9)


Associação da sub-rede pública.

![image (79)](https://github.com/user-attachments/assets/b9000f6c-f0ba-44a0-9eee-a6e2b5c4199e)

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

![image (80)](https://github.com/user-attachments/assets/57c2d511-8a52-4b92-ac59-ef1a6ef134a9)



Esse grupo permitirá que qualquer usuário na internet acesse a porta 80 (web) da sua instância EC2, o que é exatamente o que você quer para um servidor Apache simples nesse laboratório.

---

## 🖥️ Etapa 6 – Executar Instância EC2
Passos:
Vá para EC2 > Instâncias > Executar instância.

Preencha:
 - Nome da instância: `Web Server Run as Cloud`

 - AMI: `Amazon Linux 2023`

 - Tipo: `t2.micro`

 - Selecione a VPC: `lab-vpc`

 - Sub-rede: `lab-subnet-public`

 - Atribuir IP público: ✅ Habilitado

 - Grupo de segurança: selecione `Web Security Group`

Role até "Dados do usuário" e insira o script:

```
#!/bin/bash
# Use this for your user data (script from top to bottom)
# install httpd (Linux 2 version)
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "<h1>Welcome to Web Server Run as Cloud from $(hostname -f)</h1>" > /var/www/html/index.html

```
Clique em Executar instância.

📸 
Tela de configuração com os dados da instância.

![image (81)](https://github.com/user-attachments/assets/828797f4-ea33-4a88-b095-615314abdc53)

Network settings 

![image (83)](https://github.com/user-attachments/assets/74d30e72-1760-4951-869d-67d0cb3691c9)



Campo “Dados do usuário” com o script preenchido.

![image (84)](https://github.com/user-attachments/assets/5dc3f7e0-8b23-430d-82f0-46afffb9bf41)

Para esta atividade, como o foco é lançar uma instância EC2 que execute um servidor web acessível via HTTP (porta 80) e não exige conexão SSH para fins de teste, você pode escolher a opção:

✅ Proceed without key pair

![image (68)](https://github.com/user-attachments/assets/b0ba195f-b75c-4b89-943e-99a80fcdbf17)

---

## 🌐 Etapa 7 – Verificar Web Server
Passos:
Vá até a instância EC2 e aguarde até o status: 2/2 checks passed.

 - Copie o IP público da instância.

 -  No navegador, acesse: `http://[IP_PÚBLICO]`

📸 
EC2 com status "2/2 checks passed".

IP público copiado.

![image (85)](https://github.com/user-attachments/assets/7fc6a25d-0be8-431e-8c87-d34a82bee76b)


Página no navegador exibindo “Servidor Web Run as Cloud”.

![image (86)](https://github.com/user-attachments/assets/836027b3-4e2c-48fc-b03b-538d5b8e4d84)



❗ Se a página não abrir:

- Verifique se a instância está em execução e com o status "2/2 checks passed"
- Confirme que o grupo de segurança está permitindo tráfego na porta 80
- Tente acessar usando `http://<IP_PÚBLICO>` (sem "https://")

---
## 📸 Registrar Evidências 

### VPC criada com sucesso

### Sub-rede pública criada

### Internet Gateway associado

### Tabela de rotas com acesso à internet

### Security Group configurado

### Instância EC2 lançada

### Página web acessada com sucesso

---
## ✅ Conclusão
### Neste laboratório do desafio AWS VPC + EC2 + Web Server Run as Cloud, você construiu uma infraestrutura de rede personalizada e lançou uma aplicação web simples em instância EC2, utilizando apenas recursos elegíveis ao Free Tier da AWS.

### Ao final da atividade, você foi capaz de:
 - Criar uma VPC personalizada com bloco CIDR 10.0.0.0/16

 - Configurar uma sub-rede pública com acesso à internet

 - Associar um Internet Gateway e criar uma tabela de rotas

 - Criar e aplicar um Security Group com permissão HTTP (porta 80)

 - Lançar uma instância EC2 t2.micro com Apache Web Server

 - Validar o funcionamento do servidor acessando uma página de boas-vindas no navegador

🚀 Com isso, você deu um passo importante na prática com redes e instâncias na AWS, entendendo os componentes fundamentais da arquitetura em nuvem.

 ## Esse é o tipo de conhecimento essencial para quem deseja evoluir no caminho de certificações como:

##  AWS Cloud Practitioner

##   AWS Solutions Architect – Associate
---
## 👍 Parabéns! Você concluiu com sucesso o Laboratório desafio aws vpc ec2 web server run as cloud!

## 🧹 Limpeza dos recursos
Após concluir a atividade, delete os seguintes recursos para evitar cobranças:

 - Instância EC2

 - Security Group (se não reutilizável)

 - Internet Gateway

 - Tabela de rotas

 - Sub-rede

 - VPC

---
## ⚠️ Após a exclusão, revise o Painel de Faturamento da AWS e o AWS Cost Explorer para garantir que não há recursos ativos gerando custos.
## 📢 Compartilhe seu progresso!
### Marque a comunidade Run as Cloud no LinkedIn https://www.linkedin.com/company/run-as-cloud/?viewAsMember=true
Também marque os organizadores Grupos de Estudos :

Heberton Geovane https://www.linkedin.com/in/heberton-geovane/

Maik Biazi https://www.linkedin.com/in/maik-biazi-47b9291a5/

Michel Ernesto https://www.linkedin.com/in/mernesto/




