# Infraestrutura para alta disponibilidade

**Alta disponibilidade** precisa de um template (imagem) para replicar.

## Preparando para o Auto Scaling

Como a instância no EC2 foi criada com o banco de dados, e agora temos o banco mysql no RDS, precisamos separar 

  - Deixar a máquina no EC2 SEM o banco de dados
  
  - O banco ficará apenas no RDS

Conectado na máquina em EC2 e verificando o banco na porta 3306:
	
	$ netstat -ltun  

	Conexões Internet Ativas (sem os servidores)
	Proto Recv-Q Send-Q Endereço Local          Endereço Remoto         Estado     
	tcp6       0      0 :::80                   :::*                    OUÇA      
	tcp6       0      0 :::3306                 :::*                    OUÇA      
  
**Passos a executar na instância em EC2:**

1) Preparar a máquina

```
	# Parar o banco
	$ sudo systemctl stop mariadb

	# Não subir mais o banco na inicialização
	$ sudo systemctl disable mariadb
	
	# Acessar a parte web
	$ cd /var/www
	$ ls
	cgi-bin  html
	
	# Criar configuração de conexão
	$ mkdir cadastro
	$ cd cadastro
	$ vi dbinfo.cadastro
	# Colar o conteúdo abaixo dentro do arquivo dbinfo.cadastro:
```

```php	

<?php

define('DB_SERVER', 'db_instance_endpoint'); # trocar 'db_instance_endpoint' pelo endpoint do banco de dados
define('DB_USERNAME', 'admin');
define('DB_PASSWORD', 'master password'); # trocar 'master password' pela senha configurada
define('DB_DATABASE', 'cadastro');

?>
```

```
	$ ls -l
	total 4
	-rw-rw-r-- 1 ec2-user ec2-user 190 jul 12 06:31 dbinfo.cadastro

	# Criando a página de teste para acessar o banco de dados
	$ cd ..
	$ cd html
	$ vi index.php
	# Colar o conteúdo dentro do arquivo index.php:
	
```	
	
```php
<?php include "../cadastro/dbinfo.cadastro"; ?>
<html>
<body>
<h1>Cadastro WEB</h1>
<?php

  /* Connect to MySQL and select the database. */
  $connection = mysqli_connect(DB_SERVER, DB_USERNAME, DB_PASSWORD);

  if (mysqli_connect_errno()) echo "Failed to connect to MySQL: " . mysqli_connect_error();

  $database = mysqli_select_db($connection, DB_DATABASE);

  /* Ensure that the EMPLOYEES table exists. */
  VerifyEmployeesTable($connection, DB_DATABASE);

  /* If input fields are populated, add a row to the EMPLOYEES table. */
  $employee_name = htmlentities($_POST['NAME']);
  $employee_address = htmlentities($_POST['ADDRESS']);

  if (strlen($employee_name) || strlen($employee_address)) {
	AddEmployee($connection, $employee_name, $employee_address);
  }
?>

<!-- Input form -->
<form action="<?PHP echo $_SERVER['SCRIPT_NAME'] ?>" method="POST">
  <table border="0">
	<tr>
	  <td>NAME</td>
	  <td>E-MAIL</td>
	</tr>
	<tr>
	  <td>
		<input type="text" name="NAME" maxlength="45" size="30" />
	  </td>
	  <td>
		<input type="text" name="ADDRESS" maxlength="90" size="60" />
	  </td>
	  <td>
		<input type="submit" value="Add Data" />
	  </td>
	</tr>
  </table>
</form>

<!-- Display table data. -->
<table border="1" cellpadding="2" cellspacing="2">
  <tr>
	<td>ID</td>
	<td>NAME</td>
	<td>E-MAIL</td>
  </tr>

<?php

$result = mysqli_query($connection, "SELECT * FROM EMPLOYEES");

while($query_data = mysqli_fetch_row($result)) {
  echo "<tr>";
  echo "<td>",$query_data[0], "</td>",
	   "<td>",$query_data[1], "</td>",
	   "<td>",$query_data[2], "</td>";
  echo "</tr>";
}
?>

</table>

<!-- Clean up. -->
<?php

  mysqli_free_result($result);
  mysqli_close($connection);

?>

</body>
</html>


<?php

/* Add an employee to the table. */
function AddEmployee($connection, $name, $address) {
   $n = mysqli_real_escape_string($connection, $name);
   $a = mysqli_real_escape_string($connection, $address);

   $query = "INSERT INTO EMPLOYEES (NAME, ADDRESS) VALUES ('$n', '$a');";

   if(!mysqli_query($connection, $query)) echo("<p>Error adding employee data.</p>");
}

/* Check whether the table exists and, if not, create it. */
function VerifyEmployeesTable($connection, $dbName) {
  if(!TableExists("EMPLOYEES", $connection, $dbName))
  {
	 $query = "CREATE TABLE EMPLOYEES (
		 ID int(11) UNSIGNED AUTO_INCREMENT PRIMARY KEY,
		 NAME VARCHAR(45),
		 ADDRESS VARCHAR(90)
	   )";

	 if(!mysqli_query($connection, $query)) echo("<p>Error creating table.</p>");
  }
}

/* Check for the existence of a table. */
function TableExists($tableName, $connection, $dbName) {
  $t = mysqli_real_escape_string($connection, $tableName);
  $d = mysqli_real_escape_string($connection, $dbName);

  $checktable = mysqli_query($connection,
	  "SELECT TABLE_NAME FROM information_schema.TABLES WHERE TABLE_NAME = '$t' AND TABLE_SCHEMA = '$d'");

  if(mysqli_num_rows($checktable) > 0) return true;

  return false;
}
?> 
```

```
	$ ls -l
	total 4
	-rw-rw-r-- 1 ec2-user ec2-user 3064 jul 12 06:39 index.php                	
```

2) Testar se a aplicação (EC2) está rodando ok

  - Acessar no navegador com o ip público (deve visualizar a página index.php)
  
  - Inserir alguns cadastros no formulário exibido
  
3) Dando sequência, após salvar a página index.php...

```
	# Deixar conteúdo da pasta www/ no padrão usuário 'ec2-user' e grupo 'apache'
	$ cd ..
	$ cd ..
	$ ls
	account  adm  cache  db  empty  games  gopher  kerberos  lib  local  lock  log  mail  nis  opt  preserve  run  spool  tmp  www  yp

	# visualizando como está www/
	$ ls  -l
	drwxr-xr-x  5 ec2-user apache   49 jul 12 06:22 www

	# -R para aplicar de forma recursiva
	$ chown -R ec2-user:apache www/
	$ cd www/html/
	
	# agora está com usuário 'ec2-user' e grupo 'apache'
	$ ls -l
	-rw-rw-r-- 1 ec2-user apache 3064 jul 12 06:39 index.php
```

4) Pausar a instância que será gerada a imagem	
	$ sudo shutdown -h now

5) Cria a imagem

  - Selecionar a instância em EC2 \ Actions \ Image \ Create Image

  - Nomear a imagem
    - webCadastro
	
  - Create Image
  
  - Visualizar a imagem recém criada em Imagens \ AMIs \ Owned by me
    
	- Ocupa volume (EBS) também, está contando na cota
	
  - Selecionar a imagem e criar com "Launch"
    - Nos passos de criação, incluir security group para acesso remoto (ssh), web (para o apache) e default (para acessar o RDS)
	
  - Após ser criada, nomea-la
    - Pode excluir a anterior (que deu origem a esta)
	
  - Testar no navegador acessando com o ip público, se carrega ok a página de acesso ao banco de dados (index.php)	