---
layout: post
---

```
docker images // show all local images

docker build -t <image_name>:<tag> <directory>
// Builds to your local repo.

docker run --name <container_name> -p 8080:8080 -d <image_name>:<version(opt)>

docker exec -it container-name command //it is a combination of -i and -t

docker rm <container-name>

docker inspect <container_name>

docker tag original_image new_image_alias
```
