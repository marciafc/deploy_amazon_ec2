# Escalabilidade e alta disponibilidade

**ELB** (Elastic Load Balance)

  - Distribuir o tráfego entre diferentes destinos

  - Balancear tráfego TCP (por exemplo, portas diferentes da 80 e 443)
  
Referência oficial sobre [Elastic Load Balancing](https://aws.amazon.com/pt/elasticloadbalancing)  

Custo: [Definição de preço do Elastic Load Balancing](https://aws.amazon.com/pt/elasticloadbalancing/pricing)
 
## Load Balancing

<img src="./img/diagrama_load_balancing.png">

**1) Parar a instância** criada anteriormente ("webCadastro")

**2) Criar o Load Balancer**

Em Load Balancing \ Load Balancers \ Create Load Balancer \ Application Load Balancer HTTP HTTPS (clicar em "Create") 

  - Nome: lb-webCadastro
  
  - Scheme: internet-facing (virado para a internet)
  
  - IP address type: Ipv4
  
  - Load Balancer Protocol ("Add listeners")
  
    - HTTP  80
	
	- HTTPS 443
	
  - VPC (rede dentro da AWS - default)
  
  - Availability Zones (tudo será feito dentro dessas zonas de disponibilidade)
    
	- Selecionar duas subnets daquela VPC 
	
	- Next

  - No "Step 3: Configure Security Groups"
  
    - Criar um security group específico para o load balancer (marcar opção "Create a new security group")
	  
	  - Security group name: lb-acesso-web
	  
	  - definir types: HTTP na porta 80 | HTTPS na 443
	  
	  - Next

  - No "Step 4: Configure Routing"
  
    - target group (onde o load balancing olha - são as intâncias que precisam estar agrupadas)
	  
	  - Name: tg-cadastroWeb
	  
	  - Marcar: Instance
	 
      - Heath check (check se o root traz alguma resposta)
    
	  - Advanced health check settings
	
	    - Health threshold: 5 (número de tentativas de check que considera para alterar o status de uma máquina para OK)
	  
	    - Unhealth threshold: 5 (número de tentativas de check que considera para alterar o status de uma máquina para NOK)
	  
	    - Timeout: 5 segundos (limite para aguardar resposta do check, se não responder, vai considerar como falha)
	  
	    - Interval: 30 segundos (intervalo que faz o check da instância)
	  
	    - Success code: 200 (se retonar 200, considera OK)
	  
	  - Next
	  
  - No "Step 5: Register Targets" (associar o **grupo do auto scaling** e esse grupo que irá gerenciar as instâncias)

    - Irá apontar o serviço que irá escalar as máquinas, não para uma instância específica
	
	- Manter em branco por enquanto
	
	- Next
  
  - Step 6: Review
  
    - Create
	
**3) Configurar o Auto Scaling Group**

[Amazon EC2 Auto Scaling ajuda a manter a disponibilidade de seus aplicativos](https://console.aws.amazon.com/ec2autoscaling)

Amazon **CloudWatch**: habilitar **políticas de escalabilidade** e **monitorar métricas** para seus grupos de **Auto Scaling** e **instâncias EC2**.

Assim que o load balancer estiver criado (aparece em Load Balancing \ Load Balancers), segue...

  - Em Auto Scaling \ Launch Configuration (**primeiro criar setup do auto scaling, para depois lança-lo**)
      
	- Nomear o setup
	  - Name: as-config-webCadastro	  	  
      - Selecionar imagem criada anteriormente ("webCadastro") 		
	  - Tipo de instância: t2.nano
	  - Selecionar os security group já existente de acesso-remoto (ssh), acesso-web (Apache) e "default" para comunicar com RDS na mesma rede	  
	  		  
	- Selecionar chave
	
	- Clicar em "Criar launch configuration"		
		  
  - Em Auto Scaling \ Auto Scaling Groups \ clicar em "Create Auto Scaling Group" (**agora sim, criar o grupo do auto scaling**)
      
	- Group Name: as-group-cadastroWeb
	
	- No "Modelo de execução", marcar "as-config-webCadastro" (é o setup do auto scaling criado no passo anterior) \ Next
	
	- Tela "Definir configurações"	\ Rede
	  - VPC: deixar a VPC default (é a que está sendo utilizada)
	  - Subnet: as mesmas subnets selecionadas quando criou o load balancer no campo "Availability Zones" 
	    - Tem que ser as subredes que fazem parte do grupo
	 - Next					
		
	- Tela "Balanceamento de carga - opcional"
	  - Marcar "Habilitar balanceamento de carga"
	    - Application Load Balancer ou Network Load Balancer
	  - Em "Escolher um grupo de destino para seu load balancer": selecionar tg-cadastroWeb (criado no passo "Step 4" - ao definir o grupo do load balancer)
	  - Health Check Type: Marcar "ELB" (Elastic Load Balance).
	    - A opção "EC2" testa a integridade da instância, se está running, stopping, se está ligando/desligando
	    - Health Check Grace Period: para fins de teste, pode alterar para 30 segundos 	
		- Next
	
	- Tamanho do grupo - opcional
	  - Informar as capacidades (desejada, mínima, máxima) de instâncias
      - Next \ Next \ Next \ Criar o grupo de Auto Scaling	 
	    - Vai subir as instâncias
	
**Sobre criação do grupo de auto scaling é necessário:** 

   - Antes da criação do grupo de auto scaling, criar um template de configuração (launch config)
   - Configurar pelo menos duas sub-redes que devem coincidir com o **ELB**
   
**4) Testar o ambiente de produção**

  - Na listagem de grupos de auto scalling ("Auto Scaling \ Auto Scaling Groups"), selecionar o grupo criado ("as-group-cadastroWeb")
    - Na aba "Gerenciamento de Instâncias" serão listadas "x" instâncias (o número de instância configuradas na imagem) e para cada, como está seu status (Health Status)
	- Na aba "Monitoramento" é possível acompanhar o monitoramento das máquinas
	
 - O ip de chegada não é mais o da instância, mas sim o do load balancer
   - Load Balancing \ Load Balancer
   - Selecionar o load balancer na listagem, obter o "DNS name" na aba "Description" (abrir este no navegador para testar)
     - O load balancer neste momento está mandando cada requisição para uma das instâncias criadas
	 - Inserir algum registro para verificar se está salvando no RDS
	 
	- Ir no Dashboard do EC2 (Instâncias em execução), terminar uma das instâncias, e na sequência recarregar a página no navegador
	  - Verificar se outra instância foi criada(quem gerencia isso é o grupo do auto scaling)
	  
**Checks possíveis de serem configurados no grupo de auto scaling:**

 - EC2 (estado da instância EC2): stopping, terminated, etc...
 - ELB (teste da aplicação: porta 80 (HTTP) está respondendo?

**5) Configurando domínio e políticas de auto scaling**

Domínio:

  - Associar um nome ao "DNS name" do load balancer
  
  - Criar conta no [Freenom](https://www.freenom.com/pt/index.html)
    - Criar domínio
	- Criar o **CNAME** (um "apelido") para o "Public DNS (IPv4)": ec2-9-99-999-99.compute-1.amazonaws.com
    - Freenom: sem custo, um pouco lenta a interface do gerenciamento
	
	- Outra opção de para conseguir CNAME free: [Dynu](https://www.dynu.com/en-US)
	  - Assim ec2-9-99-999-99.compute-1.amazonaws.com FICA http://cadastro.usuario.webredirect.org
	  
	- Testar no navegador ou linha de comando
	
		$ dig http://cadastro.usuario.webredirect.org
	  
Políticas de auto scaling:

  - Auto Scaling \ Auto Scaling Groups \ Selecionar o grupo de auto scaling ("as-group-cadastroWeb") \ Escalabilidade automática \ Adicionar política

  - Criar política para que quando a utilização de CPU chegar a 60%, suba uma nova instância

    - Name: CPU
    - Metric type: Average CPU Utilization
    - Target Value: 60
    - Instances need: 60 segundos to warm up after scaling (valor de 60 seg para teste)
  
Também baixa a instância se tiver abaixo de 60% de CPU

Não esquecer de ajustar em Actions \ Edit \ Max (**aumentar o máximo de instâncias**) 

Exemplo de cenário: Min: 2, Max: 6 
  - Se houver uma carga maior (ex. Black Friday), ocorrerá política de "CPU" e serão alocadas dinamicamente mais máquinas (até 6 instâncias)