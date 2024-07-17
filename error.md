
**What you did**

create a `se.dockerfile` file and add the following replacing `YourApiKey` with speakeasy api key

```docker
FROM alpine:3.19
WORKDIR /app
RUN apk add bash go curl unzip sudo nodejs npm
RUN curl -fsSL https://raw.githubusercontent.com/speakeasy-api/speakeasy/main/install.sh | sh;
ENV GOPATH=/root/go
ENV PATH=$PATH:$GOPATH/bin
ENV SPEAKEASY_API_KEY=YourApiKey
```

build image will install latest cli version

```html
docker build -f se.dockerfile -t seimage .
```

- get the `pestore31.yml`  from [https://editor-next.swagger.io/](https://editor-next.swagger.io/) under **File → Load Example → OpenAPI 3.1  Petstore**
- create a folder called `app` and add the `pestore31.yml`
- replace the servers url in `pestore31.yml`

```html
servers:
  - url: https://petstore31.swagger.io/api/v31
```

generate SDK 

```html
docker run --platform linux/amd64 --rm -v "./app:/app" seimage speakeasy generate sdk --schema /app/petstore31.yaml --lang python --out /app/se
```


[https://gist.github.com/TristanSpeakEasy/07d971ed2b484e7583804dd120ba2d1b](https://gist.github.com/TristanSpeakEasy/07d971ed2b484e7583804dd120ba2d1b)

- Modify generated `app/se/.speakeasy/gen.yml`  and change `templateVersion: v1` to `templateVersion: v2`
- Change the additonal dependencies section to (or remove) in `app/se/.speakeasy/gen.yml` ( if this is not done an error `validation error: gen.yaml validation error for python: additionalDependencies.extraDependencies.dev must be a string` will occur)

```html
additionalDependencies:
  dev: {}
  main: {}
```

Regenerate the SDK 

```html
docker run --platform linux/amd64 --rm -v "./app:/app" seimage speakeasy generate sdk --schema /app/petstore31.yaml --lang python --out /app/se
```

**What you expected to happen**

![](img/0.png)

**What actually happened.**

The following error occurs

![](img/1.png)

**Workaround** 

downlaod and modify the install script from [https://raw.githubusercontent.com/speakeasy-api/speakeasy/main/install.sh](https://raw.githubusercontent.com/speakeasy-api/speakeasy/main/install.sh) to install cli version 1.321.0 instead of latest 

build image with modified script 

```html
FROM alpine:3.19
WORKDIR /app
RUN apk add bash go curl unzip sudo nodejs npm
COPY install.sh /app/install.sh
RUN chmod +x /app/install.sh && /app/install.sh
ENV GOPATH=/root/go
ENV PATH=$PATH:$GOPATH/bin
ENV SPEAKEASY_API_KEY=YourApiKey
```

```html
docker build -f se.dockerfile -t seimage .
```

generate sdk 

```bash
docker run --platform linux/amd64 --rm -v "./app:/app" seimage speakeasy generate sdk --schema /app/petstore31.yaml --lang python --out /app/se
```

Modify generated `app/se/.speakeasy/gen.yml`  and change `templateVersion: v1` to `templateVersion: v2`

regeneration seems to works even without modifying the `additionalDependencies` in `app/se/.speakeasy/gen.yml`

```docker
  additionalDependencies:
    dependencies: {}
    extraDependencies:
      dev: {}
```

regenerate sdk

```html
docker run --platform linux/amd64 --rm -v "./app:/app" seimage speakeasy generate sdk --schema /app/petstore31.yaml --lang python --out /app/se
```

![](img/2.png)