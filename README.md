# A tornado based sample python webserver in a docker image

`web-server.py` is a sample python application that hosts a simple webserver that responds to `Get` requests http requests on `localhost:8888` with the hostname.

To run the webserver manually,

1. Install python versions `2.7` or `3.3+`. 
2. Install Tornado - `pip install tornado`
3. Run `python web-server.py`
4. Try accessing <a href="http://localhost:8888/" target="_blank">http://localhost:8888</a> from your browser.

Sharing and running such a simple app like this `web-server` requires many manual steps.
Imagine packaging a real-life webserver with a lot of web content.

Now, let's see how docker can help.
Take a look at [the Dockerfile](./Dockerfile) before you begin.
Docker allows for stacking of images on top of each other.
Our docker image will be built on top of an existing docker image `library/python` which has python pre-installed.

1. Install Docker following [these instructions](https://docs.docker.com/engine/installation/).

2. Execute `docker build -t "py-web-server:v1" .` to build a docker image with web-server packaged inside of it.

3. Run the webserver
```shell
docker run -d -p 8888:8888 -h my-web-server py-web-server:v1
```

The webserver and all its dependencies including `python` and `tornado library` have been packaged into a single docker image that can now be shared with everyone.
`py-web-server:v1` docker image will function the same way on all docker supported OSes (OS X, Windows & Linux).

Try accessing the webserver again - <a href="http://localhost:8888/" target="_blank">http://localhost:8888</a>

Now let's upload the docker image to your private image repository in Google Cloud Registry (`gcr.io`).

1. Store the GCP project name in an environment variable
```shell
export GCP_PROJECT=<your-project-name>
```

2. Rebuild the docker image with an image name that includes `gcr.io` project prefix.
```shell
docker build -t "gcr.io/${GCP_PROJECT}/py-web-server:v1" .
```

3. Push the image to `gcr.io`
```shell
gcloud docker push -- gcr.io/${GCP_PROJECT}/py-web-server:v1
```

The image is now private and people with access to `$GCP_PROJECT` used above can only access it. 

Now, let's make the image accessible to everyone.
Google Container Registry stores its images on Google Cloud storage.
Let's update the permissions on Google Cloud Storage to make our image repository publically accessible.

```shell
gsutil defacl ch -u AllUsers:R gs://artifacts.${GCP_PROJECT}.appspot.com
gsutil acl ch -r -u AllUsers:R gs://artifacts.${GCP_PROJECT}.appspot.com
gsutil acl ch -u AllUsers:R gs://artifacts.${GCP_PROJECT}.appspot.com
```

The docker image can now be executed from any machine by running the following command
```shell
docker run -d -p 8888:8888 -h my-web-server gcr.io/${GCP_PROJECT}/py-web-server:v1
```

Detailed Dockerfile reference is available [here](https://docs.docker.com/engine/reference/builder/).
