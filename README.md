# Using OpenShift
# Create the Go lang builder image
The following command will create a builder image named golang-centos7.

$ oc new-build https://github.com/tinhan/s2i-go.git --name=golang-s2i
$ oc logs -f bc/golang-s2i

# Create Builder for app (Stage as build)

oc new-build golang-s2i~https://github.com/tinhan/go-restful-api-example --name=builder-<<apiname>>
oc logs -f bc/builder


# Generated artifact is located in /opt/app-root/src/go/src/main/main (Final image copy only runtime file)
oc new-build --name=runtime \\ 
   --docker-image=scratch \\ 
   --source-image=builder-<<apiname>> \\ 
   --source-image-path=/opt/app-root/src/go/src/main/main:. \\ 
   --dockerfile=$'FROM scratch\nCOPY main /main\nEXPOSE 8080\nENTRYPOINT ["/main"]'

oc logs -f bc/runtime

# Deploy and expose the app once built
oc new-app runtime --name=golang-api
oc expose svc/golang-api
