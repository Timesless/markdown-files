



### 常用命令

``` shell
docker images

docker ps -a

docker run -it -p 80:80 -v /d/data:/usr/data imageID --name containerName /bin/bash

docker exec containerID -it /bin/bash

docker update containerID --restart=always

docker restart containerID

docker logs containerID

docker stats
```

