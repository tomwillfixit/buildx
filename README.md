# Using Docker Buildx to create multi arch python apps

Following these instructions : https://docs.docker.com/language/python/build-images/

We will build a basic python app, build a multi arch image, push to docker registry.

docker build -t helloworld .

Let's start the app and verify it works :

docker run --publish 5000:5000 helloworld

Check architecture of the image we just built : docker inspect helloworld |jq -r '.[].Architecture'

Let's tag the image and push to the public registry : 

```
Login to DockerHub : docker login

docker tag helloworld tomwillfixit/helloworld:python-3.10

docker push tomwillfixit/helloworld:python-3.10
```

See image in DockerHub (Image in email shows architecture)

![default](images/dockerhub1.png)

Ok we built an image for the amd64 architecture using Docker Desktop for MacOS.

Now we want to build an image from the same source code but an image that will run on ARM based hardware such as AWS Graviton2/3.

Introducing Docker Buildx : https://docs.docker.com/buildx/working-with-buildx/

It's like docker build on steroids.

Magic command time : docker run --privileged --rm tonistiigi/binfmt --install all

What does this command do? It's all explained here : https://docs.docker.com/buildx/working-with-buildx/#build-multi-platform-images

tl;dr to build multi-arch images buildx uses QEMU emulation and this command will download some QEMU binaries and other things.

Output looks like :
```

latest: Pulling from tonistiigi/binfmt
2a625f6055a5: Pull complete 
71d6c64c6702: Pull complete 
Digest: sha256:8de6f2decb92e9001d094534bf8a92880c175bd5dfb4a9d8579f26f09821cfa2
Status: Downloaded newer image for tonistiigi/binfmt:latest
{
  "supported": [
    "linux/amd64",
    "linux/arm64",
    "linux/riscv64",
    "linux/ppc64le",
    "linux/s390x",
    "linux/386",
    "linux/mips64le",
    "linux/mips64",
    "linux/arm/v7",
    "linux/arm/v6"
  ],
  "emulators": [
    "qemu-aarch64",
    "qemu-arm",
    "qemu-mips64",
    "qemu-mips64el",
    "qemu-ppc64le",
    "qemu-riscv64",
    "qemu-s390x"
  ]
}


```

Building : docker buildx build --platform linux/amd64,linux/arm64/v8 --tag tomwillfixit/helloworld:python-3.10 .

At this point we have built an image with the "python-3.10" label that will run on amd64 and arm64.

Where the heck is this new image?

```
docker images |grep helloworld
helloworld                                                                latest                                     fa8e8485a9d9   24 minutes ago   126MB                               
tomwillfixit/helloworld                                                   python-3.10                                fa8e8485a9d9   24 minutes ago   126MB
```

The image is stored in the cache of the builder that we created earlier. The builder was called "demo" and we can view the image in there.

docker buildx imagetools inspect tomwillfixit/helloworld:python-3.10

Ok let's build again and automatically push our new multi-arch image : docker buildx build --platform linux/amd64,linux/arm64/v8 --push --tag tomwillfixit/helloworld:python-3.10 .

![multi](images/dockerhub2.png)
