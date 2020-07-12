# Banco de dados no Amazon RDS

## Dica

É possível "pinar" os serviços mais utilizados no topo do console AWS

  - Clicar no pino

  - Arrastar os serviços que serão pinados no topo (cabeçalho)

## Instância para o banco de dados

**RDS** ([Relational Database Service](https://aws.amazon.com/pt/rds) é gratuito por 12 meses se for do tipo t2.micro

Preparar uma máquina já com o mysql

Acessar o serviço [RDS](https://console.aws.amazon.com/rds) \ Create database
  
  - Marcar "Standard Create"
  
  - Marcar o "MySQL"
  
  - Em "Templates", selecionar "Free tier" (sem custo)
  
  - Cadastrar senha para o usuário do banco de dados em "Master password"
  
  - DB instance size: db.t2.micro
  
  - Database authentication: Password authentication
  
  - Additional configuration: 
    
	- Enable automatic backups(habilita backup automático)
		
	- Backup retention period (frequência do backup)
	
	- Backup window (janela do backup - pode programar o horário que será realizado o backup)

  - Create database

**VPC** é a rede virtual que a AWS fornece (rede que o banco será criado está relacionado a região selecionada)
 
 - observar se a máquina que está a aplicação web está na mesma VPC que o banco de dados

## Criando o database

Para acessar o banco de dados: 
 
  - Obter o endpoint do banco de dados

    - Acessar o dashboard do RDS \ DB Instances\ Clicar na instância \ Connectivity & security \ Endpoint & port

A máquina que for acessar o banco de dados, deve ter nos seus security groups o "default" (para ter acesso a mesma rede que foi criada)

  - Considerando que a máquina que tem o banco e a que tem o Apache foram criados na mesma rede

    - Acessar o EC2 e incluir o security group "default"
  
Conectando na máquina com a aplicação web (EC2):

	Selecionar a instância \ Connect

	ssh -i "aws-marcia.pem" root@ec2-9-999-9-999.compute-1.amazonaws.com
	
	Aparece root como usuário, pois foi criado a partir de uma imagem, mas o usuário é o ec2-user
	
	Trocar para:
	
	ssh -i "aws-marcia.pem" ec2-user@ec2-9-999-9-999.compute-1.amazonaws.com
	
	$ mysql -u admin -h <endpoint do banco de dados> -p 
	
Criando database	
	
	MySQL [(none)]> show databases;
	
	MySQL [(none)]> create database cadastro;
	
	MySQL [(none)]> show databases;