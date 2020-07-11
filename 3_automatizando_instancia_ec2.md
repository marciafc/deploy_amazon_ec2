# Automatizando a instância EC2

Criando uma instância predefinidas pela Amazon ou customizada pelo Devops

## Criando uma instância customizada

**Imagens predefinidas disponibilizadas pela AWS**

Ao clicar em "Launch instance", selecionar "AWS Marketplace"

  - Usar uma AMI da AWS Marketplace
  
  - Importante: observar se o software que terá na máquina irá gerar custo!
  
**Gerar a própria imagem customizada**

- Pode escolher imagem e instalar manual. 

- Ou via script:
  
  1) Launch instance
  
  2) Selecionar imagem e dar next até chegar a tela "Step 3: Configure Instance Details"
  
      2.1) Enable termination protection: MARCAR (não permite dar um destroy na máquina)
	
      2.2) User data: colar o conteúdo do arquivo script.sh	
	
      2.3) Next até a tela "Step 6: Configure Security Group"
	
	    - Definir o grupo de acesso
	  
	    - Next...
	  
      2.4) Selecionar a chave no passo "Step 7: Review Instance"	
	
	
**script.sh** (não precisa colocar **sudo** em nenhum comando, pois o script será executado como root)

```shell

#!/bin/bash
#Atualizando os pacotes 
# Passando -y para que quando peça a confirmação, já tenha a resposta como argumento
yum update -y

#Configurando os repositórios
amazon-linux-extras install -y lamp-mariadb10.2-php7.2 php7.2

#Instalando o apache e o mysql
yum install -y httpd mariadb-server

#inicialização automática
# "start" inicializa, "enable" sobe automaticamente sempre que inicializar a máquina
# httpd: apache
# mariadb: mysql
systemctl start httpd
systemctl enable httpd 
systemctl start mariadb
systemctl enable mariadb

#Ajustando o permissionamento
# usuário "ec2-user" vai para o grupo "apache"
# Tudo que está em  "/var/www" vai passar para usuário "ec2-user" e grupo "apache"
usermod -a -G apache ec2-user
chown -R ec2-user:apache /var/www	
```  

## Testando a instância e ajustando as regras de acesso

1) Na lista de instâncias, nomear a instância recém criada
	web-dev

2) Selecionar a instância \ Actions \ Instance Settings \ Get System Log
  - Mostra o log da instalação
  
3) Conectar na máquina (Connect \ pegar comando ssh \ executar no shell

Verificar o Apache e o banco de dados:

	$ netstat -ltun

	Porta 3306 respondendo: banco OK
	
	Porta 80 respondendo: Apache OK
	
	Conexões Internet Ativas (sem os servidores)
	Proto Recv-Q Send-Q Endereço Local          Endereço Remoto         Estado     
	tcp6       0      0 :::3306                 :::*                    OUÇA      
	tcp6       0      0 :::80                   :::*                    OUÇA      
	

Verificar permissão para o usuário (ec2-user) e o grupo (apache):

	$ cd /var
	$ ls -l
	
	drwxr-xr-x  4 ec2-user apache   33 jul 10 23:58 www	

4) Criar um Security Group para liberar a porta 80

	Security group name: acesso-web
	
	Description: acesso-web
	
	Inbound (inserir duas roles)
	
		Type: HTTP  | Origem: Qualquer lugar 0.0.0.0/0,::/0
		
		Type: HTTPS | Origem: Qualquer lugar 0.0.0.0/0,::/0
		
		--> IPv4 e IPv6: 0.0.0.0/0,::/0
	
	Clicar em "Criar grupo de segurança"
	
	Nomear na lista de Security Group: acesso-web
	
	Na instância "web-dev" adicionar este Security Group
	  - Actions \ Networking \ Change Security Groups

5) Copiar o ip público (IPv4 Public IP) da instância e acessar no navegador (deve aparecer uma página de teste do Apache)

Essa instância pode servir para virar uma imagem.