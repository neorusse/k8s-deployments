### 1.0 Dockerfile for Cloudbuild
```
FROM golang:1.14-alpine AS build-app

RUN apk add --update --no-cache gcc git build-base

WORKDIR /src/
COPY main*.go go.* /src/
RUN CGO_ENABLED=0 go build -o /bin/demo

FROM scratch
COPY --from=build-app /bin/demo /bin/demo
ENTRYPOINT ["/bin/demo"]
```

In a multi-stage Dockerfile as shown above, to build only the first stage called: build-app, run:
```
$ docker image build --target build-app --tag demo:test .
```
The --target build-app argument tells Docker to only build the part in the Dockerfile
under FROM golang:1.11-alpine AS build-app and stop before moving on to the next step.

Assuming the pipeline completes successfully, Cloud Build will automatically publish
the resulting container image to the registry. To specify which images you want to
publish, list them under images in the Cloud Build file:
images:
- gcr.io/${PROJECT_ID}/demo:${COMMIT_SHA}

## Deploying from a CD Pipeline
When you commit changes and push it the repo, that Git push uses the cloudbuild.yaml file to trigger a Build, run Tests, and Publish the final Con‐
tainer to the GCR Registry. At this stage, you’re ready to deploy that container to Kuber‐
netes.

For this example we will imagine there are two environments:

    - one for Production
    - one for Staging
  
We will deploy them into separate Namespaces:

    - Staging-demo and 
    - Production-demo

We will configure Cloud Build to deploy to staging when it sees a Git tag containing
staging, and to production when it sees production. This requires a new pipeline, in
a separate YAML file, cloudbuild-deploy.yaml. Here are the steps.

Once you have created the trigger for the staging tag, go ahead and try it out by
pushing a staging tag to the repo:

```
$ git tag -f staging
$ git push -f origin refs/tags/staging
    Total 0 (delta 0), reused 0 (delta 0)
    To github.com:domingusj/demo.git
    * [new tag]
    staging -> staging
```

If all goes as planned, Cloud Build should successfully authenticate to your GKE clus‐
ter and deploy the staging version of your application into the staging-demo name‐
space.
You can verify this by checking the GKE dashboard (or use helm status).
Finally, follow the same steps to create a trigger that deploys to production on a push
to the production tag.



