# Spring-MicroServices1

*************************************************************************************************************************
# ConfigServer
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
# Service Discovery(Eureka):
*************************************************************************************************************************************
    Every Microservice registers itself with Eureka which means eureka knows the changes in port no but the name of microservice stays 
    
    same so we can use microservice name to get details of port no from Eureka.

1. First of all we will create a new spring boot project.
	
	a).  Add dependency (eurka-server).
 	
	b). Add @EnableEurekaServer.
	
	Add below in application.properties.
	
	c)  spring.application.name=Eureka server.port=5555
    
    	d) becuase we only have one eureka 
	
		eureka.client.fetch-registry=false
		
	e) eureka.client.register-with-eureka=false 
	
	as if we have multiple eureka's then every eureka act as client to another
	
	f)  eureka.client.service-url.defaultZone=http://localhost:5555/eureka
	
2. Now we have to register microservices with Eureka.
	
	a)  Add dependency  spring-cloud-starter-netflix-eureka-client
	
	b) @EnableDiscoveryClient In All Microservices in application
	
3. We have to add Information about Eureka In Github Repo In application.properties so that Every MS knows where to find Eureka.
   
   eureka.client.service-url.defaultZone=http://localhost:5555/eureka
   
4. Ribbon With Eureka 

    a) Here we will use Name of MicroService to get all instances of Microservice.
    
    b) PlanDTO planDTO=template.getForObject("http://PLAN"+"/plans/"+custDTO.getCurrentPlan().getPlanId(), PlanDTO.class);

    c) List friends1=template.getForObject("http://FRIEND"+"/customers/"+phoneNo+"/friends", List.class); 
    
    d)Replicas must have different instance id's  eureka.instance.instance-id					
    
    =${spring.cloud.client.hostname}:${spring.application.name}:${spring.application.instance_id:${random.value}}
    
    
**************************************************************************************************************************
# Adding Resilience Using Hystrix:

Service A contacts service B but due to some network issue or server is slow service A is unable to contact Service B.

If we keep sending the requests to B thinking that it will start working after some time, that is insanity. 

We need to add resilience to our application such that the application can deal with such error situations.

A try-catch block can handle errors. But if a request is repeatedly causing an error, should the request even be continued to be sent?

Let's look at CASCADING FAILURE:

Let us say that Service D has become slow. Thus the requests to service D start queuing up.

A  ->  B  -> C -> -> -> -> D

More arrows means more requests:

Because of this latency, the requests start queuing up in all the related services slowing everything down. This is a cascading failure.

A  -> ->  -> -> ->  B -> -> -> ->  -> C -> -> -> -> D

Solution:
1. The better approach would be that if a particular service is taking more time than usual, then don’t send any more requests to that 

service. Stop requests to that service.

This prevents an increase in the slowing down of other services.

A -> B -> C (stop) D

2. This Above is similar to Cirucit Break Pattern when high voltage fuse causes the circut open saving all appliances.

 when the numbers of failures in a given time frame are more. Hystrix uses the Fail Fast approach. It is better to fail fast than to 
 
 fail big time later.
 
 3. After opening the Circuit, Hystrix will attempt to close the circuit again
 
 4. The error threshold, waiting time, retry attempts, etc are all configurable in Hystrix.
 
 Fallback:
 
 1.Hystrix allows you to mention any alternate piece of code that you wish to run if a service is down. 
 
 2.Obviously you don’t get the same result as you wish you had. But, providing some form of data instead of an error is better.
 
 Fallback executes when:

An error occurs

A timeout occurs

Circuit opens

*********************************************************************************************************************

 How to Configure Hystrix in MicroService?
 
 1. Add Dependency in A of   (spring-cloud-starter-netflix-hystrix).
 
 2. Add @EnableCircuitBreaker in Application.java file of Microservice A.
 
 3. Add @HystrixCommand(fallback="FallbackMethodName") and implement what you want to do when original method fails.
 
 4. Now time to Add Configuration for Hystrix in application.properties
  
    min 4 requests  with in 10 sec must be sent  and 50 % of the request fails then circuit breaker kicks in
    
    a) hystrix.command.default.circuitBreaker.requestVolumeThreshold=4
      
    b)  hystrix.command.default.metrics.rollingStats.timeInMilliseconds=10000
   
    c)  hystrix.command.default.circuitBreaker.errorThresholdPercentage=50
   
    d)  hystrix.command.default.circuitBreaker.sleepWindowInMilliseconds=60000
    
    Once circuit is open then for how many time it must be in open state and after that it should try to send request

    When Cirucit is open it will only go To Fallback Method ! After 60 sec's it will test for only 1 request if fails then again opens 
    
    th circuit for 60 seconds.
    
    e) Default time out for hystrix is 1 sec. 1000 ms.
    
******************************************************************************************************************************
# Synchronous Communication:

Microservice A call B and C and both these calls are independent.when calls are idependent we can make calls asynchronously.

It will save time for us.

*****************************************************************************************************************************
# API Gateway(Zuul):
*****************************************************************************************************************************
1. Zuul act as API Gateway. All requests are routed through zuul.

2. Zuul also connects to Eureka to Find the Microservices.

3.  In an API gateway pattern, you have an API gateway server that comes in between the client and the services.

*****************************************************************************************************************************
Configuration:

1. Create new Spring boot Project.

	 a)Add spring-cloud-starter-netflix-zuul dependency.

	 b) Add Cloud Config Client Dependency.

	 c) Add Eureka Discovery Client.
 
2. Add annotations:

	a) Add @EnableZuulProxy for Zuul
	
	b) Add @EnableDiscoveryClient for contacting Eureka
	
3. bootstrap.properties

	Add properties for Config Server for contacting like in all other microservices.

4. application.properties

	server.port=3333
spring.application.name=ZuulServer

zuul.routes.customer_profile.path=/customers/*
zuul.routes.customer_profile.strip-prefix=false
zuul.routes.customer_profile.service-id=CUSTOMER

eureka.client.service-url.defaultZone=http://localhost:4444/eureka

Zuul automatically uses Ribbon.

*******************************************************************************************************************************
# Feign Client:

We have been use Rest Template fot taling to other microservices.Problems with Rest Template:

1. You have to be aware of the various methods of the Rest Template API to use it.

2. You need a separate bean for Load balancing

3. You need a separate service for Circuit breaker

4. The header details of a request from Zuul are not forwarded to the other microservices using RestTemplate

What Is Feign Client?

Feign is a declarative Client From Netflix.At runtime, Feign will create an implementation for our interfaces automatically. 

Thus with minimal code and self made interfaces, we can have greater control over how one microservice communicates with the other.
**********************************************************************************************************************************
Configuration:

1. First Add Dependency for Feign Client: 

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-feign</artifactId>
    <version>1.4.7.RELEASE</version>
</dependency>

2.@ EnableFeignClients in A 

3. Create new Interface in Controller of A with annotation @FeignClient

4. @Autowire Interface bean whose implementation will be provided at run time and replace resttemplate

List<Long> friends1=friendFeign.getSpecificFriends(phoneNo);
	
*******************************************************************************************************************
# Slueth(Tracing)

1. Spring Cloud Sleuth is one of the projects under Spring Cloud which allows us to trace a request.

2. Sleuth adds two things: a traceId and a spanId in our logs. TraceId is a unique ID generated for every request. 

3. Span ID is a unique ID generated for the span or path of a single microservice. So if a request flows through two microservices,

   it will have one TraceId and two SpanId's.
   
   Configuration: Add In All MicroServices
   
 
			<artifactId>spring-cloud-starter-sleuth</artifactId>
			
********************************************************************************************************************
