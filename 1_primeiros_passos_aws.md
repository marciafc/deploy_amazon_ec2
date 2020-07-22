# Primeiros passos na AWS

Nível gratuito da AWS  - [ler](https://aws.amazon.com/pt/free/?all-free-tier.sort-by=item.additionalFields.SortRank&all-free-tier.sort-order=asc)
  - Sempre gratuito
  - 12 meses gratuitos
  - Testes

Preço do Amazon EC2 - [ler](https://aws.amazon.com/pt/ec2/pricing/on-demand/)

Console [AWS](https://console.aws.amazon.com/console)

## Dicas de segurança e controle de gastos

- Segundo fator de autenticação
  - Acessar [IAM](https://console.aws.amazon.com/iam/home)
  - Habilitar MFA em sua conta raiz \ Gerenciar MFA
  - Autenticação multifator (MFA) \ Ativar MFA 
  - Marcar "Dispositivo MFA virtual"
  - Instalar no smartphone um dos apps aceitos (por exemplo, Google Authenticator)
  - Escanear o QRCode e adicionar os códigos gerados pelo app
  
- [Alertas de uso do nível gratuito usando o Orçamentos da AWS](https://docs.aws.amazon.com/pt_br/awsaccountbilling/latest/aboutv2/tracking-free-tier-usage.html#free-budget)

- [Criando um Alarme de custos na AWS](https://www.youtube.com/watch?v=s4i8CP8SeuA&feature=youtu.be)
  
## Criando a primeira instância EC2

EC2 = Elastic Compute Cloud

Painel [EC2](https://sa-east-1.console.aws.amazon.com/ec2/v2/home)

As instâncias podem ser criadas em diversas regiões (data centers da Amazon)

Em geral aparece nestas regiões (e tendem a ser mais baratas):
  - Leste dos EUA (Norte da Virgínia) us-east-1
  - Leste dos EUA (Ohio) us-east-2

Criar instância
  - EC2 \ Instâncias em execução
  - "Launch instance"
  - Escolher qual tipo de máquina
    - Sem custo: máquinas com label "Free tier eligible"
      - Máquinas "Amazon Linux" - Red Hat compilado já para Amazon, já tem algumas ferramentas/integraçoes que auxiliam
    - Clicar em "Select"
  - Em "Step 2: Choose an Instance Type" selecionar configuração
    - Sem custo: máquinas com label "Free tier eligible"
  - Next \ Step 3: Configure Instance Details 
  - Next \ Step 4: Add Storage
  - Next \ Step 5: Add Tags
  - Next 
  - Step 6: Configure Security Group
    - Deve ficar marcada a opção: **Create a new security group** (se ainda não tem um grupo, se já foi criado, marcar "**Select an existing security group**")	
    - Nomear o "**Security group name**" = acesso-remoto
	- Description = acesso-remoto
	- Source: My IP   (só meu IP pode acessar - cuidar se o IP for dinâmico, vai parar de funcionar... há outras formas de restringir acesso)
  - Clicar em "Review and Launch"	
  - Clicar em "Launch"
  - Criar uma chave para fazer o acesso da máquina
    - Selecione "**Create a new key pair**" (se ainda não tem - caso já tenha, selecione "**Choose an existing key pair**") 
	- Nomear "Key pair name" = aws-marcia (ou selecionar a chave, caso já tenha)
	- Se estiver criando uma nova chave, efetuar seu download (salvar em local seguro - se perder, não acessa mais a máquina)
  - Clicar em "Launch instances" (a instância será provisionada)
  
  
**AWS Marketplace**: possui instância EC2 com softwares pré-instalados e configurados, como por exemplo, WordPress, MySQL

**t2.micro** e **t2.large** são **tipos de instâncias**, cada nome corresponde a uma **característica** como CPU, memória, performance...

**t2.micro** é a opção de uso dentro do **free tier**
	
