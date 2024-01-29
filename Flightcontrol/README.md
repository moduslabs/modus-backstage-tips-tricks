# Flightcontrol support for Backstage

This folder contains files that can be used to support deploying Backstage to AWS via Flightcontrol.

The Spotify Backstage documentation provides a number of examples of how to deploy Backstage. This includes
steps to deploy the codebase to AWS via Flightcontrol.

The following README and files provide an update the Dockerfile in order to make this deployment process smoother.



# Flightcontrol specific Dockerfile 

Backstage (Spotify's open source Internal Developer Platform) provides a set of documentation on using Flightcontrol to deploy to AWS Fargate:

(AWS Fargate via Flightcontrol)[https://backstage.io/docs/deployment/flightcontrol]

Using this methodology, Flightcontrol is integrated into GitHub and pulls in the Dockerfile to kick off the process of deploying to AWS. However, the issue with this approach is that you need to actually build some artifacts to copy into the image. You can see more here in the docs where the precursor to building the image is described :

(Building a Docker image)[https://backstage.io/docs/deployment/docker]

This approach creates a bit of a conundrum. Flightcontrol kicks off the deployment based on a change to a file in the repo, or via a webhook and then pulls directly from the repo. This means the build artifacts need to already be in place already in the package directory `packages/backend/dist/`.

This requires the zip files to be stored in GitHub as there is no way Flightcontrol can access a GitHub runner to grab the files direclty based on the currently documentation/methodology provided by Spotify. Flightcontrol also has no way as present of checking out the source code, running a build script (e.g. a `sh` file with the `yarn` commands) prior to creating the image.

The quick approach to remedying this is to use a multistage Docker build.

## Multistage build approach

Using a multistage build we in essence use two container images. The first is used to create the zip artifacts. The second image pulls in the artifacts so that Flightcontrol has them present when it runs the deploy process.

There are three files contained in this folder to support this process:

1. *app-config.production.flightcontrol.yaml* - this contains configuration to support the deployment. Some minor changes from the default `app-config.yaml` file include:

Switching the use of separate `host`, `port`, `user` and `password` values for a single `connectionString` value. Within Flightcontrol, when creating a database, set the output database string variable name as `POSTGRES_CON_STRING` 

Updating the `ssl:` values to include `require:true` and `rejectUnauthorized: false`


2. *Dockerfile.flightcontrol* - This is the custom Flightcontrol multistage build Dockerfile. Copy this to `<project name>/packages/backstage` and reference this in your Flightcontrol project 

3. *Dockerfile.flightcontrol.dockerignore* - A customer dockerignore file has been created to ensure that the plugins folder and packages folders aren't excluded from the build process. This should be copied into the same folder as the Dockerfile e.g. `<project name>/packages/backstage`


# Conclusion

Using these three files you will be able to quickly deploy Backstage to AWS via flightcontrol. Remember you will need to create a Postgres database via the Flightcontrol console. Post-deployment you should lock down access via AWS to your deployed Backstage instance unless you already have an Authentication mechanism integrated. 









