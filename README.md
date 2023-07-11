# testdevops
Herramientas: Minikube, docker y mvn.

## PARTE Componente Builder
Directorios k8sbuilder
Se ha utilizado la este helm chart : https://github.com/jenkinsci/helm-charts
Al arrancar el microservicio arranca el servicio Jenkis al cual podemos acceder mediante la url builder.localhost.com
Se creo una Pipeline que consta de tres Stages:

1.Stage curl: Hace un Get a una Url y obiente un Json

2.Stage Json format: Del fichero Json obtiene un valor y guarada en una variable

3.Stage Show IP: Muestra valor de esa variable

Se probo con volumen persistente pero se ha dejado configurado con un volumen NFS de tal manera que cualquier microservicio de cualquier AKS pueda acceder a la configuracion. Problema es que con Minikube al montar este tipo de volumenes daba este error:

    with: mount failed: exit status 32

  El cual parece que es debido a que hay que instalar el servicio en los nodos de nfs-utils y en Minikube no es posible.

  **Es necesario habilitar e instalar  ingress en minikube
  ** Se habilita y configura servicio ingress para acceder por builder.localhost.com
  ** Se configuran liveness y readiness, se comprueba que no hay eventos de ellos en el microservicio (añadir initialDelaySeconds: 60)
  ** se configuran los limites solicitados:
          resources:
    requests:
      cpu: "500m"
      memory: "512Mi"
    limits:
      cpu: "2000m"
      memory: "1.5Gi"

      
  
