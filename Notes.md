# Proga Web -> Curso de Microservicios
#### Realizado por: Berenice Guerrero, Marcelo Peñafiel y Daniel Seaman 

# Notas sobre el curso

## REST
Postman: POST localhost:8092/customers

```bash
{
    "id":2,
    "numberID": "12345678",
    "firstName": "Bessie",
    "lastName": "Guerrero",
    "email": "bessie.guerrero@uazuay.edu.ec",
    "photoUrl": "",
    "region": {
        "id": 2,
        "name": "Centroamérica"
    }
}
```


## Config-server 
Es un servidor de configuración centralizado -> se tienen la configuración para distintos entornos -> la configuración y su histórico se puede registrar en un controlador de versión -> así se puede centralizar la configuración de varios entornos y recargar valores de propiedades en caliente. 

### Como probar: 

En postman
	GET http://localhost:8081/customer-service/default -> Sale no autorizado
	
GET http://root:s3cr3t@localhost:8081/customer-service/default -> Tiene acceso
	
Levantamos customer-service -> GET localhost:8092/customers 

## Eureka-server

Dentro de la arquitectura -> es el discovery-service -> se hace el registro de los servicios

Es un servidor para registro y localización de las instancias de microservicios -> cada instancia(eureka-client) manda un heartbeat cada 30s 
También cada microservicio hace una copia del registro de eureka para saber que microservicios están ejecutándose en el eureka-server
Entra en modo self-preservation -> cuando eureka server no recibe el heartbeat del microservicio.

### Como probar: 
En postman
	GET http://root:s3cr3t@localhost:8081/product-service/default 

En el navegador: http://localhost:8099/ 

## Feign
Para cubrir una necesidad de comunicación en nuestro microservicio

En el shopping-service:
	Si compro algo -> necesito actualizar el stock
	Si quiero imprimir la factura -> necesito datos del cliente y del producto

Feign es una librería para generar clientes de servicio REST de manera declarativa, sin usar RestTemplate -> Se integra con Eureka para el descubrimiento-> Se integra con Hystrix para el fallback a nivel de clientes -> Se integra con Ribbon para balanceo de carga

### Como probar: 
En postman
	POST localhost:8093/invoices 
Json =
```bash 
	{
    		"numberInvoice": "002",
    		"description": "invoice office items 2",
    		"customerId": 1,
    		"items": [
        			{
            
            				"quantity": 1,
            				"price": 178.89,
            				"productId": 1
            
        			},
        			{
            				"quantity": 2,
            				"price": 40.06,
            				"productId": 3
            
        			}
    			]
}
```


	GET localhost:8091/products/1 -> el stock debe haber disminuido en 1
	GET localhost:8091/products/3 -> el stock debe haber disminuido en 2
	GET localhost:8093/invoices/2 -> ver la factura

## Hystrix

Patrón circuit-breaker -> mejora la fiabilidad del sistema -> control de latencia y tolerancia a fallos.

Dos opciones -> usar resilience4j o la versión específica para hystrix (version: '2.2.2.RELEASE')

En el navegador: http://localhost:8093/hystrix  
	
    Pasamos un endpoint donde hystrix esta enviando un stream de los eventos a través de actuator:  http://localhost:8093/actuator/hystrix.stream 
	
    En postman: GET localhost:8093/invoices/1 
	Se quita el servicio de customer
	Verificar que salga como none el customer en el invoice 
También se puede hacer pruebas de estrés, en la consola aplicar: `wrk -t4 -c5 -d5s http://localhost:8093/invoices/1`

Se está enviando en -t4(4 hilos) -c5(5 conexiones abiertas) -d5s(durante 5 seg)

## Api Gateway 

Proporciona una puerta de entrada única al ecosistema de microservicios, da enrutamiento dinámico, da monitorización y seguridad. También carga filtros en caliente.

En http://localhost:8099/ debe aparecer el Gateway.

En postman: GET localhost:8080/customers

## Actuator y Spring Boot Admin Server

Spring Boot Actuator -> libreria que da funcionalidades de monitorización y admin para apps desarrolladas con Spring Boot accesibles mediante endpoints via REST y/o JMX Bean.

Ver los Endpoints: http://localhost:8091/actuator 

Ver admin console: http://localhost:8086/applications 

## Sleuth

Librería que implementa una solución de trazado distribuido para Spring Cloud -> Da al ecosistema un mecanismo automático de identificación de peticiones ya que añade campos para identificarlas.
