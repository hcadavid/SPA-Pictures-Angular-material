### Escuela Colombiana de Ingeniería

### Construcción de Software - COSW

### Taller - frameworks MVC Web - Elementos visuales.

 

Se quiere, en el proyecto trabajado en sesiones anteriores, agregar una vista en
la que se muestren los usuarios inscritos en la aplicación de TODOs, incluyendo
sus datos básicos y su fotografía. El acceso a dicha información se hará a
través de una librería provista por un tercero.

__Parte I. Backend.__

1.  En el pom.xml agregue el siguiente repositorio de dependencias (el cual contiene la librería requerida):

	```
        <repository>
            <id>ECI internal repository</id>
            <url>http://profesores.is.escuelaing.edu.co/hcadavid/mvnmirror</url>
        </repository>
```

2.  En el mismo archivo, agregue la siguiente dependencia:

	```
        <dependency>
            <groupId>edu2.eci.cosw.stubs</groupId>
            <artifactId>FakeClientsLibrary</artifactId>
            <version>1.0</version>            
        </dependency>  
	```
 
3. Cree un API REST para el recurso 'clientes', de manera que maneje las URIs:
	* /clients   <- obtiene todos los clientes
	* /clients/{id}  <- obtiene un cliente en particular
	* /clients/{id}/picture <- obtiene la fotografía del cliente

	Para lo anterior, tenga en cuenta:
	
	*  Debe configurar la aplicación para que las anteriores rutas no sean protegidas.
	*  En la librería suministrada (como dependencia de Maven), la clase __ClientServicesStub__ tiene los siguientes métodos:


	```java
	/**
     * Consultar el conjunto completo de clientes
     * @return el conjunto de clientes
     */
    public Set<Client> getAllClients();
    
    /**
     * Consultar un determiando cliente
     * @param id identificador del cliente
     * @return el cliente
     * @throws CliendLoadException si el identificador no existe
     */
    public Client getClientById(int id) throws CliendLoadException; 
    
    /**
     * Consultar la fotografía de un determinado cliente
     * @param id el identificador del cliente
     * @return el InputStream que permite consutar los bytes de la foto
     * del cliente
     * @throws CliendLoadException 
     */
    public InputStream getClientPicture(int id) throws CliendLoadException;
	```

	* Para que un recurso REST tenga una representación diferente a la estándar (JSON), en particular un archivo, el método correspondiente del controlador REST debe retornar algo de tipo:

	```
	ResponseEntityInputStreamResource
	
	```

	Con lo anterior, el controlador puede construir la respuesta (en este caso con una imagen), usando la clase ResponseEntity:	
	```		
		ResponseEntity.ok().contentType(MediaType.parseMediaType("image/jpg")).body(new 	InputStreamResource(EL_INPUT_STREAM_DE_LA_IMAGEN));
	```
	
	Y en caso de error (por ejemplo si no se puede consultar la imagen), retornar un objeto ResponseEntity vacío que tenga el código NOT_FOUND de HTTP:
	
	```				
		return new ResponseEntity<>(HttpStatus.NOT_FOUND);
	```		

__Parte II. Front-end.__

1. Para el front-end se van a usar las siguientes librerías de JavaScript (se está actualizando la versión y se está incluyendo angular-material). Haga la actualización correspondiente con bower:

	```	
  "dependencies": {
    "angular": "~1.5.0",
    "angular-route": "~1.5.0",
    "angular-resource": "~1.5.0",
    "angular-material":"~1.0.5",
    "bootstrap-css-only": "~3.3.6"
  }
	```	
2. Rectifique en en la página html raíz se esté importando el estilo de angular-material (esto va en la sección <head> de la página). Si hay referencias a otros estilos que no se usen (por ejemplo "html5-boilerplate") elimínelos). 

	```	
<link rel="stylesheet" href="bower_components/angular-material/angular-material.css">
	```	

3. Como hay una nueva librería (con una dependencia), la misma debe cargarse desde la página principal. Agregue lo siguiente donde están las demás importaciones en la página principal:

	```	
  <!--Dependencias de Angular material-->  
  <script src="bower_components/angular-animate/angular-animate.js"></script>
  <script src="bower_components/angular-aria/angular-aria.js"></script>

  <!--Angular material-->
  <script src="bower_components/angular-material/angular-material.js"></script>	
	```	

4. Modifique la configuración del módulo principal (el definido en app.js), para que ahora importe 'ngMaterial'.
5. Cree una nueva vista (con su respectivo controlador) llamada 'Directorio de clientes', para el directorio de clientes de la aplicación.
6. En el módulo de servicios creado en el ejercicio anterior, agregue uno de tipo 'fábrica' que retorne el $recurso asociado al recurso del API REST '/clients'. Como esto retorna un arreglo de objetos, es necesario definirlo de la siguiente forma:

	```	
.factory('Clients', function($resource){
            return $resource('/clients',{},{
                get: {
                    method: 'GET',
                    isArray: true               
                }
            });
        
        })
	```	
7. En el controlador de su nueva vista inyecte el servicio antes creado ('Clients') y '$mdDialog' (un objeto de angular-material para crear ventanas emergentes).
8. En el controlador agregue un atributo (asociado a $scope) que contenga la lista de clientes. Haga que dicho atributo sea inicializado con el listado de los clientes obtenidos mediante en servicio 'Clients' inyectado.
9. En la vista cree una tabla que muestre todos los clientes, con las siguientes columnas:
	* Identificación
	* Nombre
	* Fotografía
	* Botón 'detalles'
10. Use la directiva ng-repeat para que la tabla sea poblada.
11. Haga que el campo de foto (que debería ser un elemento de tipo 'image') tenga como fuente (src) un recurso '/clientes/{id}/picture' (del API REST), donde {id} es reemplazado por el valor que tenga la tabla en la fila correspondiente. Por ejemplo, si ng-repeat usa como variable 'cl':

	```	html
<td><image src="/clients/{{cl.id}}/picture" alt="" width="80" height="100"/></td>
	```	
12. Verifique que la aplicación muestre el listado, incluyendo las fotos.
13. Revise en la documentación de Angular-material cómo abrir un diálogo: https://material.angularjs.org/latest/demo/dialog. Revise en el código javascript del ejemplo cómo se hizo la función 'showAlert'. Use esta misma función en su controlador, pero en lugar de pasar sólo el objeto 'ev', agregue como parámetro un objeto 'client'. Con esto, haga que el diálogo muestre la descripción del perfil (profileDescription).
14. Asocie con ng-click los botones dibujados en la tabla con la función antes creada (en este caso, pasando como parámetro el objeto $event y el objeto cliente -según se llame en ng-repeat-).

__Opcional__

Cambie el diálogo por uno de tipo 'custom dialog', y haga una página de detalle que muestre toda la información del cliente.
