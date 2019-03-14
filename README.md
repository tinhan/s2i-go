# Using OpenShift
Create the builder image
The following command will create a builder image named golang-centos7.

$ oc new-build https://github.com/tinhan/s2i-go.git --name=golang-s2i  
$ oc logs -f bc/golang-s2i

# Creating and running the application image
The application image combines the builder image with your applications source code, which is served using whatever application is installed via the Dockerfile, compiled using the assemble script, and run using the run script. The following command will create the application image and run it:

$ oc new-app --strategy=source --context-dir=test/test-app --name golang-s2i-api golang-s2i~https://github.com/tinhan/s2i-go.git

$ oc logs -f bc/golang-s2i-api

Using the logic defined in the assemble script, s2i will now create an application image using the builder image as a base and including the source code from the test/test-app directory.

# Creating a route for the application is needed to use it :

$ oc expose service golang-s2i-api
$ oc get route golang-s2i-api

The application, which consists of a simple static web page, should now be accessible at the url shown on last command (oc get route) :

$ curl "http://$(oc get route golang-centos7-app |awk '$1 == "golang-centos7-app" {print $2}')/"
\

# Create Builder

oc new-build golang-s2i~https://github.com/tinhan/go-restful-api-example --name=builder   
oc logs -f bc/builder


# Generated artifact is located in /opt/app-root/src/go/src/main/main
oc new-build --name=runtime \\ 
   --docker-image=scratch \\ 
   --source-image=builder \\ 
   --source-image-path=/opt/app-root/src/go/src/main/main:. \\ 
   --dockerfile=$'FROM scratch\nCOPY main /main\nEXPOSE 8080\nENTRYPOINT ["/main"]'

oc logs -f bc/runtime

# Deploy and expose the app once built
oc new-app runtime --name=golang-api
oc expose svc/golang-api
