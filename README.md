# testdevops
Herramientas: Minikube, docker y mvn.

## PARTE Componente Builder
Directorios k8sbuilder, fichero values_custom.
Se ha utilizado la este helm chart : https://github.com/jenkinsci/helm-charts
Al arrancar el microservicio arranca el servicio Jenkis al cual podemos acceder mediante la url builder.localhost.com
Se creo una Pipeline que consta de tres Stages:

1. Stage curl: Hace un Get a una Url y obiente un Json

2. Stage Json format: Del fichero Json obtiene un valor y guarda en una variable

3. Stage Show IP: Muestra valor de esa variable

Se probo con volumen persistente pero se ha dejado configurado con un volumen NFS de tal manera que cualquier microservicio de cualquier AKS pueda acceder a la configuracion. Problema es que con Minikube al montar este tipo de volumenes daba este error:

    with: mount failed: exit status 32

  El cual parece que es debido a que hay que instalar el servicio en los nodos de nfs-utils y en Minikube no es posible.

  **Es necesario habilitar e instalar  ingress en minikube.
  ** Se habilita y configura servicio ingress para acceder por builder.localhost.com.
  ** Se configuran liveness y readiness, se comprueba que no hay eventos de ellos en el microservicio (añadir initialDelaySeconds: 60).
  Se configuran los limites solicitados:
         
          
          resources:
           requests:
              cpu: "500m"
               memory: "512Mi"
            limits:
                  cpu: "2000m"
                  memory: "1.5Gi"

      
  ## Parte: Componente api-builder
  Directorios k8sbuilder, fichero values_custom.
  Se ha utilizado la este helm chart: https://github.com/bitnami/charts/tree/main/bitnami/apache.
  Se expone aplicacion en puerto 8081.
  Se añade el comando curl en la imagen mediante docker.
  Al iniciar el microservicio realiza una llamada apirest al microservicio de Jenkis y ejecuta una pipeline llamada "dtm" con el usuario testuser:

              lifecycleHooks: 
                 Example:
                     postStart:
                         exec:
                           command: ["/bin/sh", "-c", "curl -X POST -L -v --user testuser:\"${APITOKEN}\" http://builder.localhost.com/job/dtm/build"]

  En el Helm se añade la creaccion de un secret para el pass de apitoken para que sea consumida el en fichero values_custom.yaml

  Se configura ingress para acceder a la url api-builder.localhost.com

  Se crea un index.html personalizado con un codigo para desde ahi hacer llamadas a la api con un boton (no funciona, revisar funcion javascript). Se ha configurado para que use en values_custom.yaml un custom htdocs con contenido statico.
  ![Screenshot](evidencias/call_web_api_builder.jpg)
  
