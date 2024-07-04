# Anotações Spring

Precisamos mostrar ao Spring quais Beans (objetos), ele irá gerenciar. Um controller, Service, Repository... tudo isso é um Bean. Um objeto passa a ser Bean assim que passamos as anotações.

- @Component - Todas as classes são Component. Essa é uma anotação mais genérica;
```java
@Component
public class Product {

 private String name;
 private BigDecimal value;

 //... getters and setters
}
```
- @Repository - Classe onde terá lógicas de negócio do banco de dados (transações);
```java
@Repository
public class ProductRepository {
 // database transaction methods
}
```
- @Service - Classe de serviço, onde envolve regras e lógicas de negócio;
```java
@Service
public class ProductService {
 // business rules
}
```
- @Controller - Utilizada quando utilizamos uma aplicação que envolve camadas MVC (model view controller);
```java
@Controller
public class ProductController {
 // ... GET, POST, DELETE, UPDATE methods
}
```
- @RestController - Para API Rest. Se teremos somente endpoints, expondo a API na Web, utilizamos este.
<hr>

## <center> Core (Se divide em dois)
Tudo que envolve a base do Spring, está contida no Core, que fica dentro do projeto SpringFrameWork. Possui várias configurações prontas e também
a possibilidade de customizar e criar configurações necessárias para nossa aplicação.

### <center> 1. Beans
Agora que o Spring sabe quais classes serão Beans em virtude das anotações acima utilizadas (Esteriótipos), ele precisa saber **onde ele irá injetar essas instancias quando necessárias**.

Bom, nós utilizamos o **Objeto Service** dentro do nosso **Controller**, por exemplo. Pois assim podemos usar os métodos **findAll**, **findById**, etc.
Portanto, a classe **Controller** terá uma **dependência da classe Service**.

Precisamos **mostrar essa depência pro Spring**, para que ela saiba onde injetá-las quando necessário. Assim sendo, criaremos os **Pontos de Injeção (Dependência) de uma classe com a outra**.

```java
@Autowired
@Qualifier("parkingSpotServiceImpl")
private ParkingSpotService parkingSpotService;
```
- @AutoWired - Sempre que necessário, o Spring injetará o Bean Service, dentro do Bean Controller.

**Inicialmente, o ParkingSpotService importado acima, é uma interface contendo diversos métodos. Essa interface pode ter outras classes utilizando os seus métodos
(implementando ela). Ou seja, o Spring não consegue identificar qual Bean será injetado. Qual a solução? Usar a Anotação abaixo: @Qualifier**

- @Qualifier - Dentro dela, passamos qual Classe (que está implementando a interface/service) será utilizada.


- @Value - Algumas vezes precisamos definir algumas variáveis nas propriedades que usamos dentro do codigo. Mas ao invés
  de deixar isso fixo em um Controller, por exemplo, é uma boa práticar declarar essas propriedades dentro do arquivo properties.
  Logo, se precisarmos fazer alguma melhoria/alteração (**até mesmo sem parar a aplicação**), fica muito mais fácil. Exemplo:
```properties
app.name=Parking Control API
app.port=80
app.host=parkingcontrolapp
```

Para que isso seja exibido ao utilizarmos um método, criamos uma variável dentro da classe passando o @Value.
```java
@Value("${app.name}")
private String name;

@Value("${app.port}")
private String appPort;

@Value("${app.host}")
private String appHost;
```
Para exibir os valores, é só dar um SOUT dentro do método. [Veja aqui](https://github.com/MichelliBrito/parking-control-api/blob/8df47619b3016659e88dedb591c2f90ba25b5f4e/src/main/java/com/api/parkingcontrol/controllers/ParkingSpotController.java#L72)
<hr>

### <center> 2. Context

- @Configuration - Podemos configurar Datas de forma global, declarar Beans...
  **Sempre que o Spring iniciar a aplicação, ele irá olhar para essa classe e levará em consideração as customizações e configurações que definirmos nessa classe de Configuração.**
<hr>

- @ComponentScan - Podemos definir/excluir determinados pacotes que serão passados na anotação. Mostraremos para o Spring quais são os pacotes que ele irá fazer uma "varredura",
  buscando pelos Beans que ele irá gerenciar. **Serve só para algo customizado mesmo, pois o Spring em sí já faz uma varredura automática ao ser iniciado.**
```java
@Configuration
@ComponentScan(basePackages = "com.api.parkingcontrol")
public class BeanConfig{}
```
<hr>

- @Bean - Ao invés de declararmos pro spring usando as anotações esteriótipos, faremos de outra maneira.

  Criamos uma classe, por exemplo, chamada MyBean.

  Dentro da BeanConfig, criaremos um método que irá retornar exatamente o Objeto que criamos.
```java
@Bean
public myBean myBean() {
    return new MyBean();
}
```   
Pro Spring saber detectar que esse MyBean será um Bean gerenciado, precisamos passar @Bean.

Para podermos visualizar o que está dentro da Classe MyBean, faremos uma injeção de dependência dentro da Classe ParkingSpotController, assim como fizemos com o @Value.

Usa o Autowired para fazer a importação, e pode chamar o metodo com myBean.method(). [Veja aqui](https://github.com/MichelliBrito/parking-control-api/blob/8df47619b3016659e88dedb591c2f90ba25b5f4e/src/main/java/com/api/parkingcontrol/controllers/ParkingSpotController.java#L73C1-L74C1)

<div style="text-align: center;"> <b>Ok, isso foi um Bean criado por nós. Mas eventualmente usaremos Beans advindos de bibliotecas terceirizadas, como fazer nestes casos?</b> </div>

Dentro do nosso pom, inserimos uma dependência modelmapper, utilizada para conversões. [Veja aqui](https://github.com/MichelliBrito/parking-control-api/blob/8df47619b3016659e88dedb591c2f90ba25b5f4e/pom.xml#L38)

### Declarando Bean de uma biblioteca externa

Declaremos o Bean Modelmapper dentro da classe que iremos utilizar, mostrando que mesmo que esse Bean não tenha sido criado por nós, o Spring irá também gerenciá-lo.

```java
(dentro da classe config)
@Bean
public ModelMapper modelMapper() {
    return new ModelPapper();
}
```
E dentro da controller, que é onde iremos iniciar, faremos a injeção. [Veja aqui](https://github.com/MichelliBrito/parking-control-api/blob/8df47619b3016659e88dedb591c2f90ba25b5f4e/src/main/java/com/api/parkingcontrol/services/impl/ParkingSpotServiceImpl.java#L29)
<hr>

- @Lazy - Quando a gente não define nada na classe criada (passa somente @Component), o Spring internamente usa a "criação ansiosa". Já criando estes Beans, os deixando disponíveis para uso.

Mas podemos fazer um "start preguiçoso". **Ou seja, falar pro Spring criar esse Bean somente quando precisar**.

Passamos a anotação @Lazy e será feito dessa maneira.

E como acionar esse Bean em um momento específico pro Spring criar?

Bom, mesmo de sempre. Faremos a injeção de dependência no controller com @Autowired importanto o Bean e assim o Spring carregará o método com @Lazy. [Veja aqui](https://github.com/MichelliBrito/parking-control-api/blob/8df47619b3016659e88dedb591c2f90ba25b5f4e/src/main/java/com/api/parkingcontrol/controllers/ParkingSpotController.java#L46)

**Caso esse LazyBean não esteja importado dentro de algum método, não será inicializado pois o Spring não encontrará nenhum ponto de injeção de dependência.**
<hr>

- @Primary - Como é possivel ver no repositório desse projeto nos links acima. Nós temos uma interface chamada **ParkingSpotService**. Essa interface está sofrendo duas importações (ParkingSpotServiceImpl e ParkingSpotServiceImplV2).

Se você não especificar qual dessas o Spring irá considerar nos pontos de injeção que vamos criando, ele entrará em conflito. Precisamos declarar qual ele irá utilizar. Tipo quando passamos o arquivo no @Qualifier.

Então esse @Qualifier não irá existir. **Usaremos, portanto, o @Primary**. [Veja aqui](https://github.com/MichelliBrito/parking-control-api/blob/8df47619b3016659e88dedb591c2f90ba25b5f4e/src/main/java/com/api/parkingcontrol/services/impl/ParkingSpotServiceImpl.java#L18)

<hr>

- @Scope - Utilizaremos o Controller novamente e, nela, passaremos o @Scope.
Utilizaremos o parâmetro singleton, conforme vimos na parte de [Beans](#beans).
Em suma, o @Scope define como o Spring irá lidar com as criações dos Beans (dependendo do parâmetro).
<hr>

- @PropertySource - E se quisermos criar outro arquivo .properties com configurações customizadas?
Cabe destacar que o Sring lê a application.properties por default.

Podemos criar no package Resources um "custom.properties, por exemplo, contendo o que desejarmos.
No controller, criaremos uma várivel do tipo de conteúdo que criamos no arquivo "properties". Se for uma String, criaremos uma variável tipo String usando Anottation @Value. [Veja aqui](https://github.com/MichelliBrito/parking-control-api/blob/8df47619b3016659e88dedb591c2f90ba25b5f4e/src/main/java/com/api/parkingcontrol/controllers/ParkingSpotController.java#L61)

Mas isso não é o suficiente. O Spring precisa saber o arquivo pro Spring.
Passamos a anotação @PropertSoruce("classpath:custom.properties") [Veja aqui](https://github.com/MichelliBrito/parking-control-api/blob/8df47619b3016659e88dedb591c2f90ba25b5f4e/src/main/java/com/api/parkingcontrol/controllers/ParkingSpotController.java#L35)

- @PropertySources - Se for multiplos arquivos de .properties, usa ele para declarar todos os caminhos desejados.
<hr>

- @Profile - Imagine que temos um método que retorne a mesma coisa (objeto, por exemplo), porém com métodos diferentes e os dois estão com anotação @Bean.

Nessa situação, passaremos o @Profile dizendo qual será qual. Porém essa forma não é a ideal, ao inciarmos a aplicação teremos um conflito. O Spring não consegue saber qual perfil está ativo.

#### Jeitos de realizar
1. Na nossa application.properties, colocaremos logo no começo: ```spring.profiles.active=dev``` ou =prod (o que tiver no @Profile).


2. Ao invés de usarmos a anotação dentro no método dentro da BeanConfig, criaremos uma classe para cada Profile.
E assim sendo, declararemos o @Profile na classe em sí. [Veja aqui](https://github.com/MichelliBrito/parking-control-api/blob/8df47619b3016659e88dedb591c2f90ba25b5f4e/src/main/java/com/api/parkingcontrol/configs/DevBeanConfig.java#L9)
Na .properties mantém o que a gente quer passar no profile.active. O que muda mesmo é onde declaramos o @Profile.
<hr>

### Web
**Todas as Anottations web ficam no Controller** (EndPoints).

- @RestController - Usado para serviço RESTful = @Controller + @ResponseBody.


- @RequestMapping - Definimos qual a URI que o cliente irá enviar para acessar todos os métodos desse EndPoint.
Primeiro passamos só inicio dela, depois nos outros metodos abaixo passaremos o resto. [Veja aqui](https://github.com/MichelliBrito/parking-control-api/blob/8df47619b3016659e88dedb591c2f90ba25b5f4e/src/main/java/com/api/parkingcontrol/controllers/ParkingSpotController.java#L33)
```
Derivam do RequestMapping. Receberam uma função POST, GET, PUT ou DEL do tipo HTTP.

@PostMapping
@GetMapping - 
@PutMapping
@DeleteMapping
```

- @RequestBody - Ele vai desserializar de JSON para Objeto Java. [Veja aqui](https://github.com/MichelliBrito/parking-control-api/blob/8df47619b3016659e88dedb591c2f90ba25b5f4e/src/main/java/com/api/parkingcontrol/controllers/ParkingSpotController.java#L92)


- @PathVariable - No @GetMapping, por exemplo, nós passamos um caminho ```"/{id}"```. O PathVariable consegue recuperar essa parte da URI.
Basta passarmos nele ```(value = "id")```. [Veja aqui](https://github.com/MichelliBrito/parking-control-api/blob/8df47619b3016659e88dedb591c2f90ba25b5f4e/src/main/java/com/api/parkingcontrol/controllers/ParkingSpotController.java#L83)


- @RequestParam - Parecido com o PathVariable, dependendo do parâmetro, podemos colocar depois do primeiro parâmetro, um required = true/false. Para ser obrigatório ou não. [Veja aqui](https://github.com/MichelliBrito/parking-control-api/blob/8df47619b3016659e88dedb591c2f90ba25b5f4e/src/main/java/com/api/parkingcontrol/controllers/ParkingSpotController.java#L133)


- @CrossOrigin - É onde habilitamos o cors. É onde determinamos as origens/domínios que irão acessar a nossa API. Se colocar "*" será all.
maxAge = tempo máximo de resposta. [Veja aqui](https://github.com/MichelliBrito/parking-control-api/blob/8df47619b3016659e88dedb591c2f90ba25b5f4e/src/main/java/com/api/parkingcontrol/controllers/ParkingSpotController.java#L32)
Pode ser feito a nível de: classe, método e também configurações globais da aplicação.
<hr>

### Boot

- @SpringBootApplication - A classe void main irá conter essa anotação inicial. Serve para darmos o "start".
Além disso essa anotação é uma combinação de 03 outras anotações:
  1. ComponentScan - Falamos pro Spring onde será os pacotes que ele irá fazer a varredura e os Beans que ele irá gerenciar;
  2. @Configuration - Mostra que é um Bean de configuração do Spring;
  3. @EnableAutoConfiguration - Veja abaixo 👇


- @EnableAutoConfiguration - Utilizada para dizer ao Spring para ele utilizar de forma automática as suas configurações. (Tomcat, MVC, Config de registro de beans, etc...).


- @ConfigurationProperties - Lembra do @Value que passamos no Controller? Tivemos que criar variavéis para acessar esses valores que foram passados na .properties.

Nós podemos, portanto, passar esses dados do .properties para um DTO. **Ou seja, converteremos de propriedade para objeto java.**
1. Criamos uma classe para isso, chamada AppProperties [Veja aqui](https://github.com/MichelliBrito/parking-control-api/blob/8df47619b3016659e88dedb591c2f90ba25b5f4e/src/main/java/com/api/parkingcontrol/AppProperties.java#L8);
2. A anotação já vem com um parâmetro chamado "prefix = app". Isso seria um prefixo comum entre as propriedades do arquivo .properties;
   3. Criamos os métodos que iremos utilizar. Se dentro da .properties temos app.name, app.port, app.host, criaremos as variáveis;
      ```java
         private String name;
         private String port;
         private String host;
      
        //criar também os getters e setters.
      ```
   E então, dentro da classe Controller, podemos fazer o SOUT usando essa ponto de injeção. [Veja aqui](https://github.com/MichelliBrito/parking-control-api/blob/8df47619b3016659e88dedb591c2f90ba25b5f4e/src/main/java/com/api/parkingcontrol/controllers/ParkingSpotController.java#L74)
