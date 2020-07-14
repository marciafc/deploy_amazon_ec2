# AWS Command Line Interface

Acessar a AWS via CLI ao invés do Dashboard

## Instalando a AWS CLI

[Instalar do AWS CLI](https://docs.aws.amazon.com/pt_br/cli/latest/userguide/cli-chap-install.html)

Instalar a AWS CLI versão 2 no Linux

	# Instalação
	$ curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"	
	$ unzip awscliv2.zip	
	$ sudo ./aws/install
	
	$ aws --version
		
## Criar usuário para acessar o CLI da AWS

Acessar o console da AWS \ IAM \ Users \ Add user 
  - User name: aws-cli
  - Access type: Marcar "Programmatic Access" \ Next
  - "Create group" se ainda não tem
    - Preencher campo "Group name": admin
	- Selecionar Policy name: "AdministratorAccess" \ Criar um grupo
	  -> AdministratorAccess: dá direito a todos os serviços da AWS (poderia criar uma política acessar somente serviço da EC2, outra para o RDS, etc..)
  - Next \ Next \ Criar usuário
  - Clicar em "Download .csv" (contém a chave e a senha de acesso)

O arquivo .csv contém as informações a serem fornecidas no comando "**aws configure**" (estão separadas por vírgula):

	User name,Password,Access key ID,Secret access key,Console login link
	aws-cli,,<Access key>,<Secret access key>,<Console login link>
	
## Configurando a AWS CLI

	# Diretório onde é instalado (se quiser, pode adicionar no PATH: /usr/local/bin)
	$ cd /usr/local/bin	
	$ ls
	aws  aws_completer  codenation
	
	# Configurar https://docs.aws.amazon.com/pt_br/cli/latest/userguide/cli-configure-quickstart.html
	$ aws configure
	AWS Access Key ID [None]: <obter do usuário que acessa o CLI da AWS>
	AWS Secret Access Key [None]: <obter do usuário que acessa o CLI da AWS>
	Default region name [None]: us-east-1  # região da AWS (estamos utilizando o default (Norte da Virgínia)
	Default output format [None]: json     # formato de output
	
## Utilizando AWS CLI - serviços EC2

[Documentação da Interface da Linha de Comando (ILC) da AWS](https://docs.aws.amazon.com/cli/index.html)

[Referência CLI](https://docs.aws.amazon.com/pt_br/cli/latest/reference) \ Available Services

  - [Comandos EC2](https://docs.aws.amazon.com/pt_br/cli/latest/reference/ec2/index.html)

  - [Comandos RDS](https://docs.aws.amazon.com/pt_br/cli/latest/reference/rds/index.html)
  
Se não tiver adicionado o caminho no PATH, acessar o diretório onde está instalado o CLI da AWS

	$ cd /usr/local/bin
	
**Lista de comandos EC2**

	$ aws ec2 help
	
**Instâncias AWS**

Listará TODAS independente de status e respectivas informações da região pré-definida

  - Para alterar a região, basta utilizar o parâmetro --region
	
	$ aws ec2 describe-instances
	
	$ aws ec2 describe-instances | more
	
Obtendo dado de uma informação específica (no caso, do PublicDnsName)

	$ aws ec2 describe-instances | grep PublicDnsName	
	
	"PublicDnsName": "ec2-9-99-999-99.compute-1.amazonaws.com",
									"PublicDnsName": "ec2-9-99-999-99.compute-1.amazonaws.com",
											"PublicDnsName": "ec2-3-99-999-99.compute-1.amazonaws.com",
											
Listar o id e o status de cada uma das instâncias (rodando, parada, etc...)

	$ aws ec2 describe-instances --query "Reservations[*].Instances[*].{Instance:[InstanceId,State]}"
	
	[
		[
			{
				"Instance": [
					"<id da instância>",
					{
						"Code": 16,
						"Name": "running"
					}
				]
			}
		]
	]	
	
Buscar informações de uma instância pelo id

	$ aws ec2 describe-instances --instance-id <id da instância>	
	
Buscar os grupos de uma instância específica

	$ aws ec2 describe-instances --instance-id <id da instância> | grep GroupName

                        "GroupName": "",
                                    "GroupName": "acesso-web",
                                    "GroupName": "acesso-remoto",
                                    "GroupName": "default",
                            "GroupName": "acesso-web",
                            "GroupName": "acesso-remoto",
                            "GroupName": "default",	

**Controlando a instância com a AWS CLI**

 	$ aws ec2 describe-instances --instance-id <id da instância> | more
 
 	# Parar a instância
  	$ aws ec2 stop-instances --instance-id <id da instância>
	
	{
		"StoppingInstances": [
			{
				"CurrentState": {
					"Code": 64,
					"Name": "stopping"
				},
				"InstanceId": "<id da instância>",
				"PreviousState": {
					"Code": 16,
					"Name": "running"
				}
			}
		]
	}

	# Start em uma instância
	$ aws ec2 start-instances --instance-id <id da instância>
	
	{
		"StartingInstances": [
			{
				"CurrentState": {
					"Code": 0,
					"Name": "pending"
				},
				"InstanceId": "<id da instância>",
				"PreviousState": {
					"Code": 80,
					"Name": "stopped"
				}
			}
		]
	}