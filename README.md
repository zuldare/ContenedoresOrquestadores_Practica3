## Persistent Volumes
1. Creación de las carpetas correspondientes a los volumenes dentro de minikube
   ```shell
   $ minikube ssh
   $ docker@minikube:~$: mkdir /db/mysql  && mkdir /db/mongo && mkdir /rabbitmq
   ```
2. Creación del persistence volume --> pv-definition.yml
   ```shell
      kubectl apply -f pv-definition.yml
   ```
   **NOTA:** Es en este fichero donde se indica la ruta donde se va a guardar la información en disco en la etiqueta _hostPath_

3. Ver información sobre los volúmenes creados
   ```shell
      kubectl get pv
   ```
   ![Resultado](./img/1_get_pv.png)

4. Creación de la reclamación del persistence volume --> pv-claim.yml
   **IMPORTANTE:** el _metadata.name_ y _storageClassName_ **DEBEN** coincidir en ambos ficheros

5. Lanzar la reclamación
   ```shell
    kubectl apply -f pv-claim.yml
   ```

6. Ver información sobre las reclamaciones de volúmenes creados
   ```shell
      kubectl get pvc
   ```
   ![Resultado](./img/2_get_pvc.png)

7. Añadir el almacenamiento en los deployments que sea necesario y establecer el _binding_ entre el volumen montado y el volumen del contenedor, en este caso en 
   * mysql.yml
   * rabbitmq.yml
   * mongodb.yml
  
   Un ejemplo sería
   ``` yaml
    spec:
      volumes:
        - name: pv-storage-mysql
          persistentVolumeClaim:
            claimName: pv-claim-mysql
      
      containers:
      - name: mysql
        image: mysql:8.0.22
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: password
        - name: MYSQL_DATABASE
          value: eoloplantsDB
        ports:
        - containerPort: 3306 
        volumeMounts:
          - mountPath: "/var/lib/mysql"
            name: pv-storage-mysql
   ```
   **NOTA:**  claimName, debe coincidir con el nombre de establecido en pv-claim.

8. Probar que todo funciona
   * Levantamos todos los deployments, pods, pv, pvc ...
     ```shell
     kubectl apply -f .
     ```
   ![Watch All](../ContenedoresOrquestadores_Practica3/img/3_watch_all.png)
   * Probamos la aplicación y creamos una serie de nuevas plantas
   ![Watch](../ContenedoresOrquestadores_Practica3/img/4_create_plants.png)

   * Borramos el pod de mysql o para mayor prueba, borramos todos.
   ```shell
   kubectl delete pod mysql-deploy-648549fcf7-7pjcq
   kubectl delete pods --all
   ```
   * Vemos que al volver a levantar los pods se mantienen las ciudades creadas anteriormente
   ![DatabaseRemains](../ContenedoresOrquestadores_Practica3/img/5_all_plants_remain.png)


## Ingress
Creo el fichero ingress con las peticiones necesarias para la práctica:
* La web del server tiene que ser accesible en **http://cluster-ip/**
* La API REST del TopoService tiene que estar accesible en **http:// cluster-i p/ toposervice**


## Network Policies
Vamos a aplicar la política de menor privilegio --> vamos a crear un fichero de "denegación" de permisos general