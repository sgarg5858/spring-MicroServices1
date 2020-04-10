# sgarg5858-spring-MicroServices1

*************************************************************************************************************************
ConfigServer
************************************************************************************************************************


Configuration for setting up ConfigServer and Using Ribbon With Eureka.

1. First step is we put all the common properties in new made github repository under file application.properties  and for storing 

   microservice specific properties we can create files with (spring.application.name).properties .
   
2. Now we have the properties in github it's time to create ConfigServer

i) Create a new Spring Boot Project With Cloud Config Dependency. It will act as Centralized Configuration.

ii) Add @EnableConfigServer in application file.

and add properties for Config Server so that ConfigServer can take all properties from Github

spring.application.name=ConfigServer
server.port=1111
spring.cloud.config.server.git.uri=https://github.com/sgarg5858/springMicroServices
spring.cloud.config.server.git.username=******
spring.cloud.config.server.git.password=******

3.
 i) When we use Cloud Config as we create application.properties in github each microservice get's two application.properties

If we want to give priority to application.properties in github then we have to call them before local application.properties.

This can be done with the help of bootstrap context which runs before application context this way we can override the local properties.

So make the new file bootstrap.properties in every microservice and write way to call ConfigServer to recieve central properties.

spring.cloud.config.uri=http://localhost:1111

Add these dependencies in every microservice for supporting cloud config

<dependencyManagement>   <dependencies>    <dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>Greenwich.RELEASE</version>
				<type>pom</type>
				<scope>import</scope>
  </dependency>  </dependencies>  </dependencyManagement>

		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-config</artifactId>
    </dependency>


ii) If ConfigServer is down then microservices can use their local properties but while getting some property which is in central 

    application will fail so it is better if microservice doesnot rather than fail at run time if ConfigServer is down.Which can be done 
    
    with this below property.
    
     spring.cloud.config.failFast=true 
     
iii) When Using Config Server Each MicroService only calls ConfigServer Once In Start and then cache all the properties.So It won't able

    to get if any changes done after the microservices has started. Ways to get Changes in Values->
    
    a) Restart the microservice (Not a practical Approach).
    
    b) @RefreshScope and Spring Boot Actuator Dependency IN Microservice and post request to localhost:PORT/actuator/refresh it will
    
      get the changes. and in application.properties  management.endpoints.web.exposure.include=*
 
iv) Usually we deploy multiple instances of ConfigServer to ensure high availbility.

That's all for ConfigServer.

********************************************************************************************************************************
Load Balancer(Ribbon):


 
    
    
    
    

