# Container Registries

# Pushing Images to a Container Registry 

- Important to choose the base image wisely for our containers.
- Use official container images from trusted container registries and development communities.
- We need a way to further distribute our work to the various target hosts that we plan to let it run on.
- The best option to distribute a container image is to push it to a container registry, and after that, let all the target hosts pull the container image and run it.

## Container registry

- Collection of container images' repositories, used in conjunction with systems that need to pull and run container images in a dynamic way.
- Repository management
- Pushing container images
- Tag management
- Pulling container images
- Authentication management

## Repository management 

- Important to manage container images through repositories. 
- Need a web interface or a command-line interface that will let us handle the creation of a sort of *folder* that will act as a repository for our container images.
- **Open Container Initiative** (**OCI**) Distribution Specification states the container images are organized in a repository that is identified by name. 
- A repository name is usually composed of a user/organization name and the container image name in this way: `myorganization/mycontainerimage`.
- The name must respect the this regular expression check:

``` 
{.programlisting .snippet-code} [a-z0-9]+([._-][a-z0-9]+)*(/[a-z0-9]+([._-][a-z0-9]+)*)* 
```

- Once we've created a repository on our container registry, we should be able to start pushing, pulling, and handling different versions (identified by a label) for our container images.

## Pushing container images 

- Pushing container images to a container registry is handled by the container tool that we are using, which respects the OCI Distribution Specification.
- The blobs, which are the binary form of the content, are uploaded first, and, usually at the end, the manifest is then uploaded. 
	- This order is not strict and mandatory by the specification, but a registry may refuse a manifest that references blobs that it does not know.
- Using a container management tool to push a container image to a registry, we must again specify the name of the repository in the form shown before and the container image's tag we want to upload.

## Tag management 

- Can store several different versions of the container images on a system's local cache or on a container registry.

- The container registry should be able to expose the feature of content discovery, providing the list of the container images' tags to the client requesting it. 
- This feature can give the opportunity to the container registry's users to choose the right container image to pull and run on the target systems.

## Pulling container images 

- The client first requests the manifest to identify the blobs, which are the binary form of the content, to pull to get the final container image. 
- The order is strict because, without pulling and parsing the manifest file of the container image, the client would not be able to know which binary data it has to request from the registry.

- Using a container management tool to pull a container image from a registry, we have to use the fully-qualified name of the image, which includes both tag and repository.

## Authentication management 

- All the previous operations may require authentication. 
- Public container registries may allow anonymous pulling and content discovery, but for pushing container images, they require a valid authentication.

- Basic or advanced features to authenticate to a container registry. 
- Client can store a token and then use it for every operation that could require it.

- OCI Distribution Specification conformance tests to run against a container registry to check if it follows the rules defined in the specification: https://github.com/opencontainers/distribution-spec/tree/main/conformance.

## Cloud-based and on-premises container registries 

### On-premises container registries 

- Often used for creating a private repository for enterprise purposes. The main use cases include the following:

- Distributing images in a private or isolated network
- Deploying a new container image at a large scale over several machines
- Keeping any sensitive data in our own data center
- Improving the speed of pulling and pushing images using an internal network

This is a non-comprehensive list of the available container registries that we can install on-premises:

- **Docker Registry**: 
	- Provides all the basic features.
- **Harbor**
	- VMware open source project. 
	- Provides high availability, image auditing, and integration with authentication systems.
- **GitLab Container Registry** 
	- Strongly integrated with and depends on GitLab.
	- Requires minimal setup.
- **JFrog** **Artifactory**
	- Containers and other artifact management.
- **Quay**:
	- Open source distribution of Red Hat's **Quay**. 
	- Fully featured web UI.
	- Service for image vulnerability scanning, data storage, and protection.
- **Zot**
	- Open-source
	- Vendor Neutral

## Cloud-based container registries 


- Ones offered by the Linux distribution are usually only available to pull images, preloaded by the distribution maintainers.



### Docker Hub cloud registry 

- Hosted registry solution by Docker Inc. 
- Hosts official repositories and security-verified images for some popular open source projects.
- Free and paid plans


### Red Hat Quay.io cloud registry 

- Hosted registry solution born under the CoreOS company, now part of Red Hat. 
- Offers private and public repositories, automated scanning for security purposes, image builds, and integration with popular Git public repositories.
- Offers paid plans to unlock additional features.

Free tier:
- Build from a Dockerfile, manually uploaded, or even linked through GitHub/Bitbucket/Gitlab or any Git repository
- Security scans for images pushed to the registry
- Usage/auditing logs
- Robot user accounts/tokens for integrating any external software
- Five private repositories
- There is no limit on image pulls

- Paid plans will unlock more private repositories and team-based permissions.

Let\'s look at the Quay.io cloud registry by creating a public repository and linking it to a GitHub repository in which we pushed a Dockerfile to build our target container image:

1. First, we need to register or log in to the Quay.io portal at https://quay.io.

 After that, we can click on the **+ Create New Repository** button in the upper-right corner:

 <figure class="mediaobject-one"> <img src="images/B31467_9_1.png" style="width:453.11249999999995px; height:365.41330645161287px;" alt="Image 1" /> </figure>

 Figure 9.1 -- Quay Create New Repository button

2. Once done, the web portal will request some basic information about the new repository we want to create:

- A name
- A description
- Public or private (we are using a free account, so public is fine)
- How to initialize the repository:

 <figure class="mediaobject-one"> <img src="images/B31467_9_2.png" style="width:460.79999999999995px; height:336.64962406015036px;" alt="Image 2" /> </figure>

 Figure 9.2 -- The Create New Repository page

 We just defined a name for our repo, `ubi8-httpd`, and we chose to link this repository to a GitHub repository push.

3. Once confirmed, the Quay.io registry cloud portal will redirect us to GitHub to allow the authorization, and then it will ask us to select the right organization and GitHub repository to link with:

 <figure class="mediaobject-one"> <img src="images/B31467_9_3.png" style="width:460.79999999999995px; height:286.9466794995188px;" alt="Image 3" /> </figure>

 Figure 9.3 -- Selecting the GitHub repository to link with our container repo

 We just selected the default organization and the Git repository we created, holding our Dockerfile. The Git repository is named `ubi8-httpd`, and it is available here: https://github.com/PacktPublishing/Podman-for-DevOps-Second-Edition/tree/main/Chapter09/ubi8-httpd

 ::: packt_tip-one **Important** **note**

 The repository used in this example belongs to the author\'s own project. You can do the same by forking the repository on GitHub and creating your own copy with read/write permissions in order to be able to make changes and experiment with commits and automated builds. :::

4. Finally, it will ask us to further configure the trigger:

 <figure class="mediaobject-one"> <img src="images/B31467_9_4.png" style="width:460.79999999999995px; height:162.7572553430821px;" alt="Image 4" /> </figure>

 Figure 9.4 -- Build trigger customization

 We just left the default option, which will trigger a new build every time a push is made on the Git repository for any branches and tags.

5. Once done, we will be redirected to the main repository page:

 <figure class="mediaobject-one"> <img src="images/B31467_9_5.png" style="width:460.79999999999995px; height:265.617679558011px;" alt="Image 5" /> </figure>

 Figure 9.5 -- Main repository page

 Once created, the repository is empty with no information or activity, of course.

6. On the left bar, we can easily access the **Build** section. It\'s the fourth icon starting from the top. In the following figure, we just executed two pushes on our Git repository, which triggered two different builds:

 <figure class="mediaobject-one"> <img src="images/B31467_9_6.png" style="width:460.79999999999995px; height:232.49627791563273px;" alt="Image 6" /> </figure>

 Figure 9.6 -- Container image build section

7. If we try clicking on one of the builds, the cloud registry will show the details of the build:

 <figure class="mediaobject-one"> <img src="images/B31467_9_7.png" style="width:460.79999999999995px; height:243.7807622504537px;" alt="Image 7" /> </figure>

 Figure 9.7 -- Container image build details

 As we can see, the build worked as expected, connecting to the GitHub repository, downloading the Dockerfile, executing the build, and finally, pushing the image to the container registry, all in an automated way. The Dockerfile contains just a few commands for installing an `httpd` server on a UBI8 base image, as we learned in *[Chapter 8](#Chapter_8.xhtml#h1_199){.chapref}*, *Choosing* *the* *Container Base Image*.

8. Finally, the latest section that is worth mentioning is the included security scanning functionality. This feature is accessible by clicking the **Tag** icon, the second from the top in the left panel:

 <figure class="mediaobject-one"> <img src="images/B31467_9_8.png" style="width:460.79999999999995px; height:234.04496253122397px;" alt="Image 8" /> </figure>

 Figure 9.8 -- Container image tags page

As you will notice, there is a **SECURITY SCAN** column (the third) reporting the status of the scan executed on that particular container image associated with the tag name reported in the first column. By clicking on the value of that column (in the previous screenshot, it is **Passed**), we can obtain further details.

We just got some experience leveraging a container registry offered as a managed service. This could make our lives easier, reducing our operational skills, but they are not always the best option for our projects or companies.

In the next section, we will explore in detail how to manage container images with Podman\'s companion Skopeo, and then we\'ll learn how to configure and run a container registry on-premises.


### Further reading 

- \[1\] OCI Distribution Specification: https://github.com/opencontainers/distribution-spec/blob/main/spec.md
- \[2\] Bcrypt description: https://en.wikipedia.org/wiki/Bcrypt
- \[3\] Docker Registry v2 API specifications: https://docs.docker.com/registry/spec/api/

