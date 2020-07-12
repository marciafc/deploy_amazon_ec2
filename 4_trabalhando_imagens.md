# Imagens e Elastic IP

## Trabalhando com imagens

Dar um stop na instância para evitar que algo fique aberto e gere uma imagem corrompida

Selecionar instância que deseja criar imagem \ Actions \Image \ Create Image
  - Image name: web-dev-template
  - Image description: web-dev-template
  - Create Image
  
A imagem criada será listada em Images \ AMIs \ Owned by me  

**Como provisionar uma instância a partir desta imagem criada**
 
 - Launch Instance
 
 - My AMIs
 
 - Select
 
 - Mesmo fluxo de criação de uma instância
   - Marcar "Enable termination protection" no "Step 3: Configure Instance Details"
   - Marcar para incluir o Security Group que libera a porta 80 (Apache) e outros que sejam necessários no "Step 6: Configure Security Group"
   - Next...
   - Nomear a nova instância
   - Acessar no navegador pelo IPv4 Public IP
   
## IP dedicado   
  
Rede e segurança \ Elastic IPs \ Alocar endereço IP elástico \ Conjunto de endereços IPv4 da Amazon \ Alocar

Agora o endereço está disponível, mas ainda não está associado à máquina

Em Ações \ Associar endereço IP elástico \ Marcar "Instância" \ Selecionar a instância em "Instância" (pode ser uma instância parada ou running) \ Associar

**Sem custo**: Se a máquina estiver rodando, tem direito a UM IP estático associado

Se tiver por exemplo, 4 IPs estáticos associados a uma única instância: primeiro IP não é cobrado e os demais sim

Se a máquina estiver parada: conta para ser cobrado pelo IP estático

"*Quando não precisar mais de um endereço IP elástico, é recomendável liberá-lo (o endereço não deve estar associado a uma instância).* 

*Você será cobrado pelo endereço IP elástico que estiver alocado para uso com uma VPC e não estiver associado a uma instância.*"

Regras de cobrança: 

  - [Por que estou sendo cobrado pelos endereços IP elásticos quando todas as minhas instâncias são encerradas?](https://aws.amazon.com/pt/premiumsupport/knowledge-center/elastic-ip-charges/) 

  - [Endereços IP elásticos](https://docs.aws.amazon.com/pt_br/vpc/latest/userguide/vpc-eips.html)

Cada região tem um [preço diferente](https://aws.amazon.com/pt/ec2/pricing/on-demand/) (Consultar tabela de valores na seção "Endereços IP elásticos")

Avaliar sempre o [custo da máquina](https://aws.amazon.com/pt/ec2/pricing/on-demand) ligada x IP estático associado 
  - **Ponto de atenção** a ser avaliado no projeto quando for começar a trabalhar na nuvem