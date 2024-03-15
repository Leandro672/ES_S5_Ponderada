# ES_S5_Ponderada
## Introdução

Neste repositório, você encontrará todos os recursos e informações necessárias para resolução da ponderada, ou seja, criar uma instância EC2, configurar um banco de dados RDS e conectar esses componentes com uma página web, permitindo operações de criação e listagem de registros nas tabelas do banco de dados.

Para demonstrar as máquinas/serviços em execução foi anexado um video com a demonstração abaixo descrevendo como o deploy foi realizado e o que cada máquina/serviço faz.

como observado no video, a partir do tutorial da ponderada e os conhecimentos adquiridos em sala, a conexão do RDS com a instancia ec2 foi realizada com sucesso

## Código

Segue a seguir o código utilizado no video para analize e correção


````
<?php include "../inc/dbinfo.inc"; ?>
<html>
<body>
<h1>Sample page</h1>
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
  $employee_age = htmlentities($_POST['AGE']);
  $employee_salary = htmlentities($_POST['SALARY']);
  $is_employed = isset($_POST['IS_EMPLOYED']) ? 1 : 0;

  if (strlen($employee_name) || strlen($employee_address) || strlen($employee_age) || strlen($employee_salary)) {
    AddEmployee($connection, $employee_name, $employee_address, $employee_age, $employee_salary, $is_employed);
  }
?>

<!-- Input form -->
<form action="<?PHP echo $_SERVER['SCRIPT_NAME'] ?>" method="POST">
  <table border="0">
    <tr>
      <td>NAME</td>
      <td>ADDRESS</td>
      <td>AGE</td>
      <td>SALARY</td>
      <td>EMPLOYED</td>
    </tr>
    <tr>
      <td>
        <input type="text" name="NAME" maxlength="45" size="30" />
      </td>
      <td>
        <input type="text" name="ADDRESS" maxlength="90" size="60" />
      </td>
      <td>
        <input type="text" name="AGE" maxlength="3" size="5" />
      </td>
      <td>
        <input type="text" name="SALARY" maxlength="10" size="10" />
      </td>
      <td>
        <input type="checkbox" name="IS_EMPLOYED" />
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
    <td>ADDRESS</td>
    <td>AGE</td>
    <td>SALARY</td>
    <td>EMPLOYED</td>
  </tr>

<?php

$result = mysqli_query($connection, "SELECT * FROM EMPLOYEES");

while($query_data = mysqli_fetch_row($result)) {
  echo "<tr>";
  echo "<td>",$query_data[0], "</td>",
       "<td>",$query_data[1], "</td>",
       "<td>",$query_data[2], "</td>",
       "<td>",$query_data[3], "</td>",
       "<td>",$query_data[4], "</td>",
       "<td>", $query_data[5] ? 'Yes' : 'No', "</td>"; // Display 'Yes' if 1, 'No' if 0
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
function AddEmployee($connection, $name, $address, $age, $salary, $is_employed) {
   $n = mysqli_real_escape_string($connection, $name);
   $a = mysqli_real_escape_string($connection, $address);
   $ag = mysqli_real_escape_string($connection, $age);
   $s = mysqli_real_escape_string($connection, $salary);

   $query = "INSERT INTO EMPLOYEES (NAME, ADDRESS, AGE, SALARY, IS_EMPLOYED) VALUES ('$n', '$a', '$ag', '$s', '$is_employed');";

   if(!mysqli_query($connection, $query)) echo("<p>Error adding employee data.</p>");
}

/* Check whether the table exists and, if not, create it. */
function VerifyEmployeesTable($connection, $dbName) {
  if(!TableExists("EMPLOYEES", $connection, $dbName))
  {
     $query = "CREATE TABLE EMPLOYEES (
         ID int(11) UNSIGNED AUTO_INCREMENT PRIMARY KEY,
         NAME VARCHAR(45),
         ADDRESS VARCHAR(90),
         AGE INT,
         SALARY DECIMAL(10, 2),
         IS_EMPLOYED TINYINT(1) DEFAULT 0
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
````
