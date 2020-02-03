# Maven Profiles - Uma Solução para Separação de Ambientes

## Como era antes dos Profiles do maven 

Durante o desenvolviomento de uma aplicação, é bem complicado isolar as configurações do ambiente em que a aplicação está. 
É bem comun ver algo mais ou menos assim :

```properties
##dev
spring.datasource.username=user
spring.datasource.password=user
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.url=jdbc:h2:file:./databaseDev

##hml
#spring.datasource.username=admin
#spring.datasource.password=admin
#spring.datasource.driver-class-name=org.h2.Driver
#spring.datasource.url=jdbc:h2:file:./databaseHomol

##prod
#spring.datasource.username=root
#spring.datasource.password=root
#spring.datasource.driver-class-name=org.h2.Driver
#spring.datasource.url=jdbc:h2:file:./databaseProd
```
_(application.properties para desenvolvimento)_

Desse modo, caso fosse necessário rodar em ambiente de homologação seria feito isso :

```properties
##dev
#spring.datasource.username=user
#spring.datasource.password=user
#spring.datasource.driver-class-name=org.h2.Driver
#spring.datasource.url=jdbc:h2:file:./databaseDev

##hml
spring.datasource.username=admin
spring.datasource.password=admin
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.url=jdbc:h2:file:./databaseHomol

##prod
#spring.datasource.username=root
#spring.datasource.password=root
#spring.datasource.driver-class-name=org.h2.Driver
#spring.datasource.url=jdbc:h2:file:./databaseProd
```
_(application.properties para Homologação)_

Mas vamos lá ... fazer as coisas desse modo não é muito saudável, principalmente em um contexto de CI e CD. Isso acabaria gerando coisas desnecessárias. Inclusive, ter essa abordagem faz sua apliação estar na ameaça de erro humano, afinal, seu estagiário não imaginaria que precisaria ficar comentando codigo para fazer deploy, não é ?

![Estagiarios](https://www.ibahia.com/fileadmin/user_upload/ibahia/2017/agosto/18/memeestagio8.jpg)

Mas calma, seu estagiário não é burro. Se ele fez algo parecido, ou exatamente isso ele apenas foi vitima do sistema... nesse caso, de um sistema mal arquitetado e quiçá um arquiteto mal intencionado.
___

## Agora, com maven profiles... A história é outra

### Ajustando os Perfis

Primeiramente teremos um arquivo para cada perfil de configuração e ambiente, nele carregaremos as variaveis que vão conter os valores que cada perfil vai esperar. Para ficar mais lúdico, vamos continuar usando o exemplo acima.

Primeiramente, vamos criar pastas no sistema, para guardar esses profiles em algum local que possa ser acessado pelo nosso querido Maven.

Assim como no codigo fonte desse projeto, eu costumo coloca-los numa pasta chamada **profiles**, que fica ao lado de **src**. E dentro dela uma pasta para cada profile (afinal, podemos ter outras configurações para aquele profile, por exemplo se formos configurar LS4J personalizado ou varios persistence.xml) 

Para o ambiente de desenvolvimento teremos:
```properties

db_username=user
db_password=user
db_driver=org.h2.Driver
db_url=jdbc:h2:file:./databaseDev
```
_(arquivo app.properties localizado na pasta **profiles/dev/** )_

Para o ambiente de Homologação teremos:
```properties

db_username=admin
db_password=admin
db_driver=org.h2.Driver
db_url=jdbc:h2:file:./databaseHomol
```
_(arquivo app.properties localizado na pasta **profiles/homolog/** )_

Para o ambiente de Produção teremos:
```properties
db_username=root
db_password=root
db_driver=org.h2.Driver
db_url=jdbc:h2:file:./databaseProd
```
_(arquivo app.properties localizado na pasta **profiles/prod/** )_

### Ajustando Application.properties

application.properties é uma parte importante do nosso sistema, é ele que carrega as configurações, e, como no caso do projeto, se voce ultiliza SpringBoot, é uma das partes mais importantes da sua aplicação.

Ultilizando as variaveis, ele ficara assim :
```properties
spring.datasource.username=@db_username@
spring.datasource.password=@db_password@
spring.datasource.driver-class-name=@db_driver@
spring.datasource.url=@db_url@
```


> _Tá mais e aí ? Somente isso para eu voltar a ter segurança na minha aplicação ?_ 

Não, ainda não... sabe porque ? Porque o maven olha para o seu pom.xml, e o maven não fez curso _"Mãe diná"_ para saber que voce está usando perfis. 

> _OK, e como eu faço o meu Maven saber que estou usando perfis ?_

Vamos resolver isso configurando o pom.xml

### Configurando pom.xml

#### Explique-me como
Antes de configurar o pom, voce pode achar chato, mas terei que explicar algumas tags, e a primeira delas é essa :

* ```<id>dev</id>```
Esta tag diz qual o nome identificador do perfil que vamos acessar, nesse caso, eu chamarei esse perfil de "dev", mas poderia chama-lo de "Batman" se eu assim o quisesse, bastaria mudar o conteudo entre as tags.
<br>

* ```<filter>profile/dev/app.properties</filter>```
Esta tag tenta encontrar arquivos com extensão **_.properties_** com variaveis declaradas para aplica-las no projeto
<br>

* ```<filters></filters>```
Bom... eu acho que essa tag é autoexplicativa mas lá vai: Ela guarda as tags ```<filter></filter>``` , normalmente, ela é colocada dentro de uma tag ```<build></build>```
<br>

* ```<profile></profile>```
Esta tag... bem... guarda as configurações do profile. Ou seja, todas as tags acima ficam dentro dele, e ele fica dentro de uma tag ```<profiles></profiles>```

#### Vamos para a prática

Levando como base o exemplo que tivemos nas ultimas partes da explicação, o pom.xml ficaria assim : 
```xml
<profiles>
    <profile>
        <id>dev</id>
        <build>
            <filters>
                <filter>profiles/dev/app.properties</filter>
            </filters>
        </build>
    </profile>
    <profile>
        <id>homolog</id>
        <build>
            <filters>
                <filter>profiles/homolog/app.properties</filter>
            </filters>
        </build>
    </profile>
    <profile>
        <id>prod</id>
        <build>
            <filters>
                <filter>profiles/prod/app.properties</filter>
            </filters>
        </build>
    </profile>
</profiles>
```

> _OK, E agora ? Meu projeto já está configurado com perfis do maven ?_

Sim, está, basta apenas que voce faça ```mvn {command} -P {id_profile}``` para conseguir rodar a sua aplicação no profile que desejou. Leve este pequeno projeto como inspiração para suas aventuras com os perfis do maven. 

Espero ter ajudado.
