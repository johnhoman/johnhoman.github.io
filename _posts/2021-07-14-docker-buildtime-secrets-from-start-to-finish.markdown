## Private Python Packages
Packaging protect artifacts into a container image has it's challenges. The
build runtime likely requires secrets to fetch protected resources, secrets
such as username and password. It's actually really easy to accidently leak
a secret into an image and can be really difficult to recover from.

Obviously we can't do something like

```Dockerfile
FROM python3.9-slim

COPY requirements.txt /requirements.txt
RUN python -m pip install -r /requirements.txt \
        --index-url https://me:f30ba0a8-61dc-445b-84e7-839028679e28@pypi.jackhoman.com/simple || true
```

Well, technically we could do this but even if we weren't going to commit it to
source control (which we are) it's still a terrible idea to put credentials
in plaintext for a number of reasons.

Running the following (replace the tag with something you can push to or
just don't run the push command)
```sh
docker build -f Dockerfile -t docker.example.com/app:1.0.0 .
docker push docker.example.com/app:1.0.0 .
```

So, the image is built and pushed, now the sensitive information in the 
`Dockerfile` can be removed and we're ready for downstream applications to 
consume the image we just built `docker.example.com/app:1.0.0`. The only
problem is the sensistive information isn't gone. We can recover it running
the following

```sh
docker history docker.example.com/app:1.0.0 --no-trunc | grep index-url
```
You'll see something that resembles the following
```console
sha256:99ecd8d3b36a3cf26163d68212497a7ea9638316d588c1bc7eff1e148644bc38   7 minutes ago   RUN /bin/sh -c python -m pip install -r /requirements.txt       --index-url https://me:f30ba0a8-61dc-445b-84e7-839028679e28@pypi.jackhoman.com/simple || true # buildkit
```
Anyone that has access to that image now has access to your package index.


## Temporary Credentials (probably an AWS example)
## Running on Github Actions
