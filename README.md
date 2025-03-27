# SESION DE LABORATORIO N° 01: PRUEBAS ESTATICAS DE SEGURIDAD DE APLICACIONES CON SONARQUBE

## OBJETIVOS
  * Comprender el funcionamiento de las pruebas estaticas de seguridad de còdigo de las aplicaciones que desarrollamos utilizando SonarQube.

## REQUERIMIENTOS
  * Conocimientos: 
    - Conocimientos básicos de Bash (powershell).
    - Conocimientos básicos de Contenedores (Docker).
  * Hardware:
    - Virtualization activada en el BIOS..
    - CPU SLAT-capable feature.
    - Al menos 4GB de RAM.
  * Software:
    - Windows 10 64bit: Pro, Enterprise o Education (1607 Anniversary Update, Build 14393 o Superior)
    - Docker Desktop 
    - Powershell versión 7.x
    - Net 8 o superior
    - Visual Studio Code

## CONSIDERACIONES INICIALES
  * Clonar el repositorio mediante git para tener los recursos necesarios
  * Tener una cuenta de Github valida. 

## DESARROLLO
### Parte I: Configuración de la herramienta de Pruebas Estaticas de Seguridad de la Aplicación
1. Ingrear a la pagina de SonarCloud (https://www.sonarsource.com/products/sonarcloud/), iniciar sesión con su cuenta de Github.
2. Ingresar a la opción My Account
   ![image](https://github.com/UPT-FAING-EPIS/lab_calidad_01/assets/10199939/bd49c592-47f5-4767-9f15-c56ad6802818)
   
3. Generar un nuevo token con el nombre que desee, luego de generar el token, guarde el resultado en algún archivo o aplicación de notas. Debido a que se utilizará
   ![image](https://github.com/UPT-FAING-EPIS/lab_calidad_01/assets/10199939/75941062-40f0-4689-8c91-6603ced490a3)
  
4. En el navegador, ingresar a la url https://sonarcloud.io/projects/create, para generar un nuevo proyecto con el nombre apibank, apunte el nombre del Project Key, que se utilizara mas adelante, luego dar click en el boton next.
   ![image](https://github.com/user-attachments/assets/95819e23-6dcf-4ca4-800d-7bfe0f686bf3)
   
5. En el navegador, finalizar la creación del proyecto, haciendo click en la opción Previous Version, y luego en el boton Create Project
   ![image](https://github.com/user-attachments/assets/7f8a6d86-f14d-4b49-882e-0d0bb35d5069)


### Parte II: Creación de la aplicación
1. Iniciar la aplicación Powershell o Windows Terminal en modo administrador 
2. Ejecutar el siguiente comando para crear una nueva solución
```
dotnet new sln -o Bank
```
3. Acceder a la solución creada y ejecutar el siguiente comando para crear una nueva libreria de clases y adicionarla a la solución actual.
```
cd Bank
dotnet new classlib -o Bank.Domain
dotnet sln add ./Bank.Domain/Bank.Domain.csproj
```
4. Ejecutar el siguiente comando para crear un nuevo proyecto de pruebas y adicionarla a la solución actual
```
dotnet new mstest -o Bank.Domain.Tests
dotnet sln add ./Bank.Domain.Tests/Bank.Domain.Tests.csproj
dotnet add ./Bank.Domain.Tests/Bank.Domain.Tests.csproj reference ./Bank.Domain/Bank.Domain.csproj
```
5. Iniciar Visual Studio Code (VS Code) abriendo el folder de la solución como proyecto. En el proyecto Bank.Domain, si existe un archivo Class1.cs proceder a eliminarlo. Asimismo en el proyecto Bank.Domain.Tests si existiese un archivo UnitTest1.cs, también proceder a eliminarlo.

6. En VS Code, en el proyecto Bank.WebApi proceder la carpeta `Models` y dentro de esta el archivo BankAccount.cs e introducir el siguiente código:
```C#
namespace Bank.Domain.Models
{
    public class BankAccount
    {
        private readonly string m_customerName;
        private double m_balance;
        private BankAccount() { }
        public BankAccount(string customerName, double balance)
        {
            m_customerName = customerName;
            m_balance = balance;
        }
        public string CustomerName { get { return m_customerName; } }
        public double Balance { get { return m_balance; }  }
        public void Debit(double amount)
        {
            if (amount > m_balance)
                throw new ArgumentOutOfRangeException("amount");
            if (amount < 0)
                throw new ArgumentOutOfRangeException("amount");
            m_balance -= amount;
        }
        public void Credit(double amount)
        {
            if (amount < 0)
                throw new ArgumentOutOfRangeException("amount");
            m_balance += amount;
        }
    }
}
```
7. Luego en el proyecto Bank.WepApi.Tests añadir un nuevo archivo BanckAccountTests.cs e introducir el siguiente código:
```C#
using Bank.Domain.Models;
using NUnit.Framework;

namespace Bank.Domain.Tests
{
    public class BankAccountTests
    {
        [Test]
        public void Debit_WithValidAmount_UpdatesBalance()
        {
            // Arrange
            double beginningBalance = 11.99;
            double debitAmount = 4.55;
            double expected = 7.44;
            BankAccount account = new BankAccount("Mr. Bryan Walton", beginningBalance);
            // Act
            account.Debit(debitAmount);
            // Assert
            double actual = account.Balance;
            Assert.AreEqual(expected, actual, 0.001, "Account not debited correctly");
        }
    }
}
```
8. En el terminal, ejecutar las pruebas de lo nostruiido hasta el momento:
```Bash
dotnet test --collect:"XPlat Code Coverage"
```
> Resultado
```Bash
Failed!  - Failed:     0, Passed:     1, Skipped:     0, Total:     1, Duration: < 1 ms
```
9. En el terminal, instalar la herramienta de .Net Sonar Scanner que permitirá conectarse a SonarQube para realizar las pruebas estáticas de la seguridad del código de la aplicación :
```Bash
dotnet tool install -g dotnet-sonarscanner
```
> Resultado
```Bash
Puede invocar la herramienta con el comando siguiente: dotnet-sonarscanner
La herramienta "dotnet-sonarscanner" (versión 'X.X.X') se instaló correctamente
```
10. En el terminal, ejecutar :
```Bash
dotnet sonarscanner begin /k:"PROJECT_KEY" /d:sonar.token="TOKEN" /d:sonar.host.url="https://sonarcloud.io" /o:"ORGANIZATION" /d:sonar.cs.opencover.reportsPaths="*/*/*/coverage.opencover.xml"
```
> Donde:
> - TOKEN: es el token que previamente se genero en la pagina de Sonar Source
> - ORGANIZATION: es el nombre de la organización generada en Sonar Source
> - PROJECT_KEY: es el nombre de la llave proyecto que previamente se genero en la pagina en Sonar Source

12. En el terminal, ejecutar:
```Bash
dotnet build --no-incremental
dotnet test --collect:"XPlat Code Coverage;Format=opencover"
```
13. Ejecutar nuevamente el paso 8 para lo cual se obtendra una respuesta similar a la siguiente:
```Bash
dotnet sonarscanner end /d:sonar.token="TOKEN"
```
14. En la pagina de Sonar Source verificar el resultado del analisis.

![image](https://github.com/UPT-FAING-EPIS/lab_calidad_01/assets/10199939/4e4892d3-71e2-4437-9713-a270ebf61b06)

15. Abrir un nuevo navegador de internet o pestaña con la url de su repositorio de Github ```https://github.com/UPT-FAING-EPIS/nombre_de_su_repositorio```, abrir la pestaña con el nombre *Settings*, en la opción *Secrets and Actions*, selecionar Actions y hacer click en el botón *New Respository Token*, en la ventana colocar en Nombre (Name): SONAR_TOKEN y en Secreto (Secret): el valor del token de Sonar Cloud, guardado previamente

![image](https://github.com/user-attachments/assets/cf0b874f-7eb9-4888-a37d-9a975316d53f)

16. En el VS Code, proceder a crear la carpeta .github/workflow y dentro de esta crear el archivo sonar.yml con el siguiente contenido, reemplazar los valores ORGANIZATION y PROJECT_KEY con los valores obtenidos de SonarCloud
```Yaml
name: Sonar Continuos Integration
env:
  DOTNET_VERSION: '8.x'                     # la versión de .NET
  SONAR_ORG: 'ORGANIZATION'                    # Nombre de la organización de sonar cloud
  SONAR_PROJECT: 'PROJECT_KEY'        # Key ID del proyecto de sonar
on:
  push:
    branches: [ "main" ]
  workflow_dispatch:

jobs:
  sonarqube:
    name: Sonarqube Analisys
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '17'
      - name: Configurando la versión de NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: ${{ env.DOTNET_VERSION }}
      - name: Restaurar los paquetes
        run: dotnet restore 
      - name: Ejecutar pruebas
        run: dotnet test --collect:"XPlat Code Coverage;Format=opencover"
      - name: Instalar Scanner
        run: dotnet tool install -g dotnet-sonarscanner
      - name: Ejecutar escaneo
        run: | 
          dotnet-sonarscanner begin /k:"${{ env.SONAR_PROJECT }}" /o:"${{ env.SONAR_ORG }}" /d:sonar.login="${{ secrets.SONAR_TOKEN }}" /d:sonar.host.url="https://sonarcloud.io" /d:sonar.cs.opencover.reportsPaths="*/*/*/coverage.opencover.xml" /d:sonar.qualitygate.wait=true
          dotnet build
          dotnet-sonarscanner end /d:sonar.login="${{ secrets.SONAR_TOKEN }}"
```

---
## Actividades Encargadas
1. Adicionar un metodos de prueba para verificar el método de crédito.
2. Adjuntar la captura donde se evidencia el incremento del valor de cobertura en un archivo cobertura.png.
3. Adicionar a la automatizacion la construcción del archivo .nuget y la publicación como paquete en su repositorio de Github
4. Adicionar a la automatizacion la generación del release de la versión 1.0.0 del nuget, debe indicar las modificaciones del paquete en base a los comentarios de los commits realizados
   
