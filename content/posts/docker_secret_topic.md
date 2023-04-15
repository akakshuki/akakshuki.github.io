---
title: "Docker secret topic"
date: 2023-04-15T11:46:00+07:00
type: "post"
tags: ["docker"]
---

# Docker Secret

When you need to provide a credential with docker file like API key, credentials, certificates. If you hard-code credentials in to your dockerfile or you .env files into the image so you are opening a un-secure door here, attacker can extract your credential or gain access. Even If you remove a file after using it dockerfile; it still exist in the image layer.

The solution is DOCKER SECRET

## Setup a demo


we have a dockerfile to run python script:
file myapp.py have only line print('hello')

```docker
FROM python:3.9-alpine
WORKDIR /app

COPY "myapp.py" .

ENTRYPOINT ["python"]
CMD ["myapp.py"]
```
 
## Adding private stuff: 

```bash
pip install --extra-index-url https://username:password@my.pypi.com/pypi privatestuff
```

## Wrong way: 

### Hard-code ❌:

You might think about hard code your credential  in your dockerfile. Do not ever do it, this is just asking a problems

```docker 
FROM python:3.9-alpine
WORKDIR /app

RUN python -m pip install --extra-index-url https://username:password@my.pypi.com/pypi privatestuff
COPY "myapp.py" .

ENTRYPOINT ["python"]
CMD ["myapp.py"]
```

### Copy file and remove ❌:  

In Docker, deleting a file does not actually remove it from the image. Since Docker cashes its layers; all previous layers are still present in the image. This means that a malicious user can easily extract your .netrc file.

```docker
FROM python:3.9-alpine
WORKDIR /app

COPY .netrc .
RUN python -m pip install --extra-index-url https://username:password@my.pypi.com/pypi privatestuff
RUN rm -rf .netrc
COPY "myapp.py" .

ENTRYPOINT ["python"]
CMD ["myapp.py"]
```


### Use  --build-arg in docker command  ❌:

This seem like a good option but it's also supper bad. 

```docker
FROM python:3.9-alpine
WORKDIR /app

ARG PYPI_USER=pypi_user
ARG PYPI_PASS=pypi_pass
ARG PYPI_URL=pypi_url

RUN python -m pip install --extra-index-url https://${PYPI_USER}:${PYPI_PASS}@${PYPI_URL} dnxlogger

COPY "myapp.py" .

ENTRYPOINT ["python"]
CMD ["myapp.py"]
```

we have three arguments here which we will provide the command bellow 

```bash
python -m pip install --extra-index-url https://${PYPI_USER}:${PYPI_PASS}@${PYPI_URL} dnxlogger
```

This will pass the arguments to our Dockerfile but it is very easy to extract them from the layers in the image.

```bash
docker history my_app --no-trunc | grep PYPI_PASS
```

with command above in docker history --> trace with PYPI password --> here we have the credential 🙂 


![Alt text](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*LvGkFsm6l-0wCv8SfvQWwA.png "Title")


## Correct way

### We will use the docker secret to secure your image ✅

Here is may have only way to secure and easy to implement. So I am a fan of .env files 
1. Simple enough. We create a file called .env and give it the following content:

```bash
export PYPI_URL=my.pypi.com/pypi
export PYPI_USER=m.huls
export PYPI_PASS=supersecretpassword
```

2. Define docker build command  

```bash
 docker build -t "my_app" --secret id=my_env,src=.env .
```

You can see the command above as: load the .env file and give it a key called my_env. We can use this key in the next step to access the .env.

3. Modify Dockerfile so that it mounts the secret

```docker 
FROM python:3.9-alpine
WORKDIR /app

RUN --mount=type=secret,id=my_env source /run/secrets/my_env \
  && python -m pip install --extra-index-url https://${PYPI_USER}:${PYPI_PASS}@${PYPI_URL} privatestuff
  
COPY "myapp.py" .

ENTRYPOINT ["python"]
CMD ["myapp.py"]

```
As you can see we just add one extra line of code to our Dockerfile. We mount a secret specifying the id my_env (which we’ve specified in step 2). Next we source the content of the .env which load its content as variables. This is why we can use the variables in the — extra-index-url as specified in the pip install.

The beauty of this approach is that the content of the .env file is only accessible in the RUN command where it’s referred in.
    
![Alt text](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*HgmhDnLxtHiAWBUI692o6Q.png "Title")


Enjoy with your docker 😌 
