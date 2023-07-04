# Intel Arc Stable Diffusion Setup with Automatic WEB-ui

My notes.

```
sudo apt update && sudo apt -y upgrade
sudo apt install docker.io

git clone https://github.com/intel/intel-extension-for-pytorch
cd intel-extension-for-pytorch
git checkout xpu-master

cd docker
./build.sh xpu-flex


IMAGE_NAME=intel-extension-for-pytorch:xpu-flex

VIDEO=$(getent group video | sed -E 's,^video:[^:]*:([^:]*):.*$,\1,')
RENDER=$(getent group render | sed -E 's,^render:[^:]*:([^:]*):.*$,\1,')

test -z "$RENDER" || RENDER_GROUP="--group-add ${RENDER}"

docker run --rm \
    -v <your-local-dir>:/workspace \
    --group-add ${VIDEO} \
    ${RENDER_GROUP} \
    --device=/dev/dri \
    --ipc=host \
    -e http_proxy=$http_proxy \
    -e https_proxy=$https_proxy \
    -e no_proxy=$no_proxy \
    -it $IMAGE_NAME bash
```

Create Dockerfile
```
FROM intel-extension-for-pytorch:xpu-flex

RUN apt update && apt -y upgrade

RUN apt install libgl1 libglib2.0-0 -y

RUN git clone https://github.com/vladmandic/automatic/
RUN cd automatic && git checkout dev
```

```
docker build . reto-ipex

IMAGE_NAME=intel-extension-for-pytorch:xpu-flex

VIDEO=$(getent group video | sed -E 's,^video:[^:]*:([^:]*):.*$,\1,')
RENDER=$(getent group render | sed -E 's,^render:[^:]*:([^:]*):.*$,\1,')

test -z "$RENDER" || RENDER_GROUP="--group-add ${RENDER}"

docker run --rm \
    -v <your-local-dir>:/workspace \
    --group-add ${VIDEO} \
    ${RENDER_GROUP} \
    --device=/dev/dri \
    --ipc=host \
    -p 7860:7860 \
    -e http_proxy=$http_proxy \
    -e https_proxy=$https_proxy \
    -e no_proxy=$no_proxy \
    -it reto-ipex:latest bash
```

inside container

```
cd automatic/
python launch.py --use-ipex --listen
```

