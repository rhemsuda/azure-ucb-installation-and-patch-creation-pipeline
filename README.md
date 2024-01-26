# Azure / UCB Installation & Patch Creation Pipeline
Azure DevOps pipeline written to manage installation and patch artifacts using the Unity Cloud Build (UCB) API for cross-platform build management

How it works:
1. Build is triggered from Azure pipelines starting the azure-build-pipeline.yml, which manages build resources and communicates with UCB to pass through relevant information about the build like commit hash & type of update (major, minor, patch).
2. UCB receives a build request and starts the build based on the commit hash. When the build is successful, a configured webhook sends the details to an Azure function, which relays them back to our pipeline, starting the azure-deploy-pipeline.yml file with the newly generated build number.
3. The deployment pipeline communicates with the database to get the most recent version of code previous to this build, as well as the newly generated code from the UCB share.
4. The contents of the build directory are hashed, sorted, and then hashed again to generate a unique hash for the contents of the build. This is stored on the in the database so client file integrity can be checked when making new connections to the server.
5. File sizes are stored in the database so capacity requirements can be clearly defined as installed files are compressed, and client needs to store uncompressed files.
6. A new installation file is created by compressing the build data. The file is uploaded to a CDN for fast public retrieval.
7. A patch artifact is generated using the bsdiff utility to create binary diff files for each file in the build, comparing previous file contents to new file contents.
8. A patch file is created by compressing the patch data. The file is uploaded to a CDN for fast public retrieval.

The launcher can then call a series of API endpoints to interact with the database, get the most recent version & it's installation/patch links. If the game isn't installed, it can use the install URL, and if the game is installed but is out of date, it can download all the patches and run bspatch to update the client.

This is demonstrated in the PlatformLauncher repository
