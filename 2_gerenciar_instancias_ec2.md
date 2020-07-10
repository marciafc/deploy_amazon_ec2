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

Para obter o comando, selecionar a instância e clicar em "**Conect**". Será exibido:

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

## Gerenciar as instâncias Amazon Linux

- Par de chaves para autenticação

- SSH é o protocolo padrão para gerenciamento remoto 

## Remover instância

Destrói a instância:

  - Selecionar a instância \ Actions \ Instance State \ Terminate
