1. Create a compose file for the stack.

`vi simple-stack.yml`

```
 version: '3'
 services:
   web:
     image: nginx
   busybox:
     image: radial/busyboxplus:curl
     command: /bin/sh -c "while true; do echo Hello!; sleep 10; done"
```
2. Deploy the stack and examine it using various commands.

```
 docker stack deploy -c simple-stack.yml simple
 docker stack ls
 docker stack ps simple
 docker stack services simple
 docker service logs simple_busybox
```
3. Modify the stack to use an environment variable.

`vi simple-stack.yml`

```
 version: '3'
 services:
   web:
     image: nginx
   busybox:
     image: radial/busyboxplus:curl
     command: /bin/sh -c "while true; do echo $$MESSAGE; sleep 10; done"
     environment:
     - MESSAGE=Hello!
```
```
 docker stack deploy -c simple-stack.yml simple
 docker service logs simple_busybox
```
4. Modify the stack to expose a port.

`vi simple-stack.yml`

```
 version: '3'
 services:
   web:
     image: nginx
     ports:
     - "8080:80"
   busybox:
     image: radial/busyboxplus:curl
     command: /bin/sh -c "while true; do echo $$MESSAGE; sleep 10; done"
     environment:
     - MESSAGE=Hello!
```
```
 docker stack deploy -c simple-stack.yml simple
 curl localhost:8080
```
5. Modify the stack to use the BusyBox service to communicate with the web service.

` vi simple-stack.yml`
```
 version: '3'
 services:
   web:
     image: nginx
     ports:
     - "8080:80"
   busybox:
     image: radial/busyboxplus:curl
     command: /bin/sh -c "while true; do echo $$MESSAGE; curl web:80; sleep 10; done"
     environment:
     - MESSAGE=Hello!
```
 `docker stack deploy -c simple-stack.yml simple`
 
 6. Delete the stack.

`docker stack rm simple`