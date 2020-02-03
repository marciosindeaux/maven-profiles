# Maven Profiles - Uma Solução para Separação de Ambientes
___

### Como era antes dos Profiles do maven 

Durante o desenvolviomento de uma aplicação, é bem complicado isolar as configurações do ambiente em que a aplicação está. 
É bem comun ver algo mais ou menos assim :

```properties
##dev
spring.datasource.username=user
spring.datasource.password=user

##hml
#spring.datasource.username=admin
#spring.datasource.password=admin

##prod
#spring.datasource.username=root
#spring.datasource.password=root
```
_(application.properties para desenvolvimento)_

Desse modo, caso fosse necessário rodar em ambiente de homologação seria feito isso :

```properties
##dev
#spring.datasource.username=user
#spring.datasource.password=user

##hml
spring.datasource.username=admin
spring.datasource.password=admin

##prod
#spring.datasource.username=root
#spring.datasource.password=root
```
_(application.properties para Homologação)_

Mas vamos lá ... fazer as coisas desse modo não é muito saudável, principalmente em um contexto de CI e CD. Isso acabaria gerando coisas desnecessárias. Inclusive, ter essa abordagem faz sua apliação estar na ameaça de erro humano, afinal, seu estagiário não imaginaria qur precisaria ficar comentando codigo para fazer deploy, não é ?

!(Estagiarios)[https://www.ibahia.com/fileadmin/user_upload/ibahia/2017/agosto/18/memeestagio8.jpg]
