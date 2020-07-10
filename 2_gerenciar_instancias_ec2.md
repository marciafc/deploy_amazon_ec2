# Gerenciando instâncias EC2

Listando as [instâncias](https://console.aws.amazon.com/ec2/v2/home?region=us-east-1#Instances:sort=instanceId)
  - Colocar um nome: clicar no ícone de lápis na coluna "Nome"
  
[Mapa da infraestrutura global](https://aws.amazon.com/pt/about-aws/global-infrastructure/?hp=tile&tile=map) da AWS  

**Regiões**: são os data centers da Amazon
  - cada região tem suas **zonas de disponibilidade** (redundância). Por exemplo, Leste dos EUA (Norte da Virgínia) **us-east-1** que tem:
    - us-east-1a
	- us-east-1b
	- us-east-1c
	- us-east-1d (incrementa a letra até completar a quantidade de zonas de disponilidade)	

[Regiões, zonas de disponibilidade e zonas locais](https://docs.aws.amazon.com/pt_br/AmazonRDS/latest/UserGuide/Concepts.RegionsAndAvailabilityZones.html)

## Acessando a máquina via ssh

Para obter o comando, selecionar a instância e clicar em "**Connect**". Será exibido:

	ssh -i "aws-marcia.pem" ec2-user@ec2-99-999-999-99.compute-1.amazonaws.com
	
Executar este comando na mesma pasta que está o arquivo **.pem** (chave gerada anteriormente - "**aws-marcia.pem**")

  - Confirmar: "Are you sure you want to continue connecting (yes/no)?" yes
  - Vai dar erro pq a chave tem que ser de leitura só para o dono

	$ ls -l
    -rw-rw-r--  1 marcialinux marcialinux       1692 jul 10 02:07  aws-marcia.pem
    
	$ chmod 400  aws-marcia.pem
	
	$ ls -l
	-r--------  1 marcialinux marcialinux       1692 jul 10 02:07  aws-marcia.pem
	
	$ ssh -i "aws-marcia.pem" ec2-user@ec2-54-197-209-34.compute-1.amazonaws.com

Feito! Está na máquina da Amazon	

**ec2-user**: usuário

**99-999-999-99**: ip dinâmico (tem como fixar)

**8ec2-99-999-999-99.compute-1.amazonaws.com**: public DNS (IPv4)

-> Quando conecta na máquina, o que aparece no terminal é o ip privado

	[ec2-user@ip-888-88-88-888 ~]   = ec2-user é o usuário | ip-888-88-88-888 é parte do DNS privado | 888-88-88-888 é o IP privado


## Gerenciar as instâncias Amazon Linux

- Par de chaves para autenticação

- SSH é o protocolo padrão para gerenciamento remoto 

## Remover instância

Destrói a instância:

  - Selecionar a instância \ Actions \ Instance State \ Terminate
  
A máquina fica em status "terminated". No próximo ciclo de atualização da Amazon ela irá sumir.  
  
## Proteção contra exclusão

Inibe que a máquina seja terminada:

1) Selecionar a instância

2) Actions \ Instance Settings \ Change Termination Protection \ Yes, Enable

É uma boa prática em máquinas em produção para evitar excluir acidentalmente.

Para desabilitar a proteção: mesmo processo e marcar "Yes, Disable"

## Comunicação das instâncias

Após conectar na máquina via ssh, atualizar os pacotes:

	$ sudo yum update

### Autorizar a comunicação entre as máquinas:

**Visualizando a regra de acesso dos Security groups **
	
	1) Acessar o EC2 Dashboards (ou Painel EC2)
	
	2) Security groups (ou Grupos de segurança)
	
	3) Selecionar o grupo de segurança que deseja visualizar as regras
	
	4) Aba "Inbound" (ou "Regras de entrada")
	
Quando criamos um Security group, também é criado automaticamente, um **Security group com o nome = "default"** (Descrição = "default VPC security group")
	
	Selecione este no passo 3 e visualize a aba "Inbound" (ou "Regras de entrada")
	
		Coluna "Origem" é a própria rede (na qual foi criada)
		
**Para liberar acesso a outra instância na mesma rede**

Máquina A (aplicação web) acessando máquina B (banco de dados):

	Selecione a instância \ Actions \ Networking \ Change Security Groups \ Marcar "default"\ "Assign Security Groups
	
	Fazer esse procedimento nas máquinas A e na B, o pacote tem que ir e voltar, senão só irá ocorrer o envio
	
	$ ping <ip privado da máquina>
	
Pode criar um security group por porta também.
	
### Visualizar quais grupos estão associados a quais instâncias:

	Acessar o EC2 Dashboards (ou Painel EC2)
	
	Instâncias em execução
	
	Selecionar instância
	
	Actions \ Networking \ Change Security Groups OU nos detalhes da instância em "Security groups"
	
## VPC

Redes dentro da AWS

**As máquinas são criadas dentro da mesma sub rede**

## Preço da instância e preço do disco 

São cobrados separados

**EBS** (Elastic Block Store): nome que a Amazon utiliza para se referir aos discos

"O nível gratuito da AWS inclui 30 GB de armazenamento, 2 milhões de E/S e 1 GB de armazenamento de snapshots com o Amazon Elastic Block Store (EBS)."
  
  -> Pode ser uma máquina de 30GB ou várias, desde que a soma entre elas, não ultrapasse esse limite de 30GB (sempre consultar se esse limite não foi alterado)
  
[Definição de preço do Amazon EBS](https://aws.amazon.com/pt/ebs/pricing/)

### Para saber quanto de disco está consumindo

Elastic Block Store \ Volumes

Se a máquina estiver parada (stop), ainda está consumindo disco (EBS), logo contabilizará para os limites da conta free e se exceder, será cobrado

[Calculadora estimativa de preço -  SIMPLE MONTHLY CALCULATOR](https://calculator.s3.amazonaws.com/index.html)

	- Preencher "Compute: Amazon EC2 Instances" e "Storage: Amazon EBS Volumes" (preço da instância + preço do disco)

[Calculadora estimativa de preço - AWS Pricing Calculator](https://calculator.aws/)