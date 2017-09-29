# Wercker Maven Step Sample

This repository contains a sample application that demonstrates how to use the Wercker Maven Step. 

The example `wercker.yml` below demonstrates the basic usage of the Wercker Maven Step.  There are a few interesting things to note:

* You can run the step more than once, as shown in the rather contrived example below of running `compile` in one step and then `install` in another.  A more realistic use of this may be to run a separate profile that performs deployment (copies built artifacts into some well known location for example).

* If you run the step more than once, you can set the `cache_repo: true` property as shown below.  This will put the Maven local repository into the Wercker cache, meaning it will be available in subsequent builds.  If you run this build twice over and pay careful attention to what Maven downloads each time, you will notice that in the first build, the first step downloads the plugins and dependencies needed to clean and compile, and the second step downloads those needed to install (and other phases in between).  The second build will not need to download anything becuase the required JARs are already in the repository. 

* The first script steps installs the extra dependencies needed to run this step, which are not in the image/box I am using here.

* If you look carefully at the output of the build, you will see that Maven is only installed if it is not already present.  The step will use Maven if it finds it installed in `/maven` but it is recommended to let the step install it itself in order to minimize image size.

``` 
box:
  id: store/oracle/serverjre
  username: $DOCKER_USERNAME
  password: $DOCKER_PASSWORD
  tag: 8

test-with-wercker-cache:
  steps:
    - script:
      name: Install pre-reqs
      code: |
        yum -y install tar gzip
    - wercker/step-maven:
      goals: clean compile
      cache_repo: true
    - wercker/step-maven:
      goals: install
      cache_repo: true
    # you should push your image now :)
    #  - internal/docker-push:
    #    repository: quay.io/myuser/myapp
    #    tag: 1.0.0

```