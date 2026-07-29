
Skaffold watches for code changes in a repo and builds, tests, and deployes and application in Kubernetes

### What questions are they asking? Headline melding.

How can I validate the structure of a container image?

How do you run tests on a new version of your application and automate the creation of a new container?

Set up an automation that watches source code for changes. If there are changes, then you will have a set of step ready to go that deploy to a K8s cluster, VMs, or your chosen Cloud provider.

Build > test > deploy

Skaffold is a CD tool for kubernetes-native applications.
Container-structure test will check and verify the structure of and image after it is built.
- can verify specific files exist
- Can run commands and verify output.
- Can verify container metadata set in a dockerfile such and environment variables, ports

Skaffold will give you a template to build and test your code in the same way. This helps produce a consistent product and makes automatic deployment easier. 

During the testing phase, 
- Unit tests
- Integration Tests
	- Does the application integrate within the rest of you environment?
- Security Scans
	- Check for known security vulnerabilities in software or imported container images. 

Artifact is then built and sent to a repo that the CD phase has access to.

The CD Takes the artifact and deploys it using one of three strategies:
- Canary
	- Like sending a canary into a cole mine. If a select group of users survive, then the code is pushed out to everyone. 
- Rolling
	- Deploys new code slowly along side of current code until all the new code is fully deployed. 
- blue-green
	- Green (new) code is tested along side of blue (current)  code. If the green code is good, then it is deployed. 

Monitoring is deployed to watch for errors, latency, etc. If a problem is detected then the code can be rolled back. 

skaffold.yaml
- Provides instructions on building, testing, and deploying an application. 
- Located in root of project
- Keep this file under version control
- Three main sections: 
	- build
		- how to build the image
	- test
		- tests to perform on the image
	- deploy
		- How to deploy to kubernetes

```yaml
kind: Config
build:
  artifacts:
    - image: project/code
  local: {}
test:
  - image: project/code
	custom:
     - command: "npm --prefix ./src test"
    structureTests:
      - ./container-structure-tool/structure-tool.yaml
deploy:
  kubectl:
   manifests:
   - kubernetes/*
```

**Build section** uses the Docker build command to create the container image. The image name is set to project/code with the image tag. This will match the image name in deployment.yaml

Skaffold uses the current Git commit hash to calculate the image tag. The tag is added to the container image name automatically and set as an environment variable called $IMAGE.

**test section**
Run any tests on the container image or application. The example file above runs a container structure test that checks the container for defects then a custom Node.js test to 

**deploy section**
Uses kubernetes manifest files in /kubernetes directory to release the deployment. A patch is placed on the current deployment and the container tag and image are replaced. 

**container-structure-test**
Run on the container after it is built and after the application has been tested. 

In this case, the test is located in the application's subdirectory /container-structure-tool/ in a file called structure-tool.yaml

```yaml
commandTests:
  - name: "telnet-server"
	command: "./telnet-server"
    args: ["-i"]
    expectedOutput: ["telnet port :2323\nMetrics Port: :9000"]
metadataTest:
  env:
    - key: TELNET_PORT
      value: 2323
    - key: METRIC_PORT
      value: 9000
  cmd: ["./telnet-server"]
  workdir: "/app
```

**commandTests**
command that runs the binary and args adds the -i option to output info to STDOUT. The output must match the "expectedOutput" value in order to pass the test. 

**metadataTest**
Checks the Docker file to make sure variables, commands, and working directory data matches. 

**skaffold run subcommand**
Build's, tests, and deploys the application. Does not watch for new code changes. 

**skaffold dev subcommand**
Same as run but watches for changes in source code and automatically runs the tests on new changes. 

Pods will quit running after you press ^C so add the --cleanup=false argument if you want it to keep running. 

Ran while in the code directory you are running the test on. 
```bash
$ skaffold dev --cleanup=false 
Listing files to watch... 
- dftd/telnet-server Generating tags... 
# Sets the container tag number, based on current git commit hash.
- dftd/telnet-server -> dftd/telnet-server:4622725 
- Checking cache... 
- dftd/telnet-server: Not found. Building 
- Found [minikube] context, using local docker daemon. Building [dftd/telnet-server]
...
Successfully tagged dftd/telnet-server:4622725
# skaffold starts the test section
Starting test...
Testing images...
=======================================================
====== Test file: command-and-metadata-test.yaml ======
=======================================================
=== RUN: Command Test: telnet-server --- PASS duration: 571.602755ms stdout: telnet port :2323 Metrics Port: :9000 === RUN: Metadata Test --- PASS duration: 0s ======================================================= ======================= RESULTS ======================= ======================================================= Passes: 2 Failures: 0 Duration: 571.602755ms

Total tests: 2 PASS Running custom test command: "go test ./... -v" ? telnet-server [no test files] ? telnet-server/metrics [no test files] === RUN TestServerRun Mocked charge notification function TestServerRun: server_test.go:23: PASS: Run() --- PASS: TestServerRun (0.00s) PASS ok telnet-server/telnet (cached) Command finished successfully.

# Starts deploy
Starting deploy... - deployment.apps/telnet-server created - service/telnet-server created - service/telnet-server-metrics created Waiting for deployments to stabilize... - deployment/telnet-server: waiting for rollout to finish: 0 of 2 updated replicas are available... - pod/telnet-server-6497d64d7f-j8jq5: creating container telnet-server - pod/telnet-server-6497d64d7f-sx5ll: creating container telnet-server - deployment/telnet-server: waiting for rollout to finish: 1 of 2 updated replicas are available... - deployment/telnet-server is ready.

Deployments stabilized in 2.140948622s **Press Ctrl+C to exit** Watching for changes...

```

Can use the debug flag if you have any errors for troubleshooting. 

skaffold will skip the build and test phase if there are no changes detected. -dirty is added to the end of the tag if the repo has uncommited changes. 

Once you make a code change, the process will be re-run automatically if the dev option is used. 

You can use the minicube tunnel command to jump to the server to test.

Grab the ip of the server:
```bash
$ minikube kubectl -- get services telnet-server
NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGE 
telnet-server LoadBalancer 10.105.161.160 **10.105.161.160** 2323:30488/TCP 6m40s
```

Verify that new pods are running:
`minikube kubectl`

Rolling back

If there is no immediate disruption in service then you can deploy a hot fix and run it through the CI/CD pipeline again. If your service is down and you just need to bring it up, then just use Kubernetes to roll back.

1. Check the rollout history:
- Kubernetes tracks deployments and saves states of previous versions.  

To check deployment history:
```bash
minikube kubectl -- rollout history deployment telnet-server
deployment.apps/telnet-server
REVISION CHANGE-CAUSE
1           <none>
2           <none>
```

Revision 2 is the current active. 

To rollback to revision 1:
```bash
minikube kubectl -- rollout undo deployment telnet-server --to-revision=1
deployment.apps/telnet-server rolled back
```

Or leave off the `--to-revision` option to just roll back to the previous version. 

Verify:
```bash
minikube kubectl -- get pods
NAME READY STATUS RESTARTS AGE telnet-server-7fb57bd65f-qc8rg 1/1 Running 0 28s telnet-server-7fb57bd65f-wv4t9 1/1 Running 0 29s
```

Enter the app to verify the change

The new revision is version 3 as listed with the `rollout history` command.

If your company focuses on Mean Time to Recovery (MTTR), this technique helps the service go back up as fast as possible from a customer's point of view. 