---
title: "Building an Elevator Twin - Part 1"
date: 2024-02-03T00:39:59+01:00
draft: false # Set 'false' to publish
tableOfContents: false # Enable/disable Table of Contents
description: 'In this series of artiicles, I share my learnings as I build a digital elevator IoT sytem using EdgexFoundry.'
categories:
  - iot
  - edgex
tags:
  - iot
  - edgex
  - edgexfoundry
---

This is the first of a multipart series. I would be sharing about an [interesting toy project](https://github.com/osifo/digital-elevator/) I'm working on - A digital twin for elevators using open source technologies.
The IoT framework I would be using for this project is [Edgex Foundry](https://www.edgexfoundry.org).

### Why Edgex Foundry.
I chose Edgex Foundry for a number of reasons:
- It is truly open source; It uses the [Apache (2.0) license](https://github.com/edgexfoundry/edgex-go/blob/main/LICENSE). This means it is completely free to use - even for commercial purposes.
- It is built with Go. For me, this is one more way to get better at the language quickly.
- It is micro-service based. This allows for modular development, so I don't need to worry much about [fragility](https://archive.blogs.harvard.edu/samuelgerges/antifragile-enterprise-microservices-architecture).
- It is containerized. As a result, running my solution on various OS/device specs quite seamless.

### A little summary about Edgex Foundry (Edgex)

Edgex is a vendor-neutral, OS anarchicture agnostic and microservice-based open source framework for the IoT edge. What this means is that it helps manage how our devices interact with the real world (via sensors and actuators) as well as how this data is processed, stored and sometimes integrated with other edge or cloud technologies. It is primarily written in Go and is being managed by the Apache Software Foundation. [This article*](https://docs.edgexfoundry.org/3.1/) by the Edgex team, provides a more exhaustive introoduction than I could ever attempt. Also, just like I did, you can very helpful information to get you started from [the official Youtube channel](https://www.youtube.com/@edgexfoundry8766).

_P.S At the time of writing this article, the most recent stable version of Edgex is 3.1(Napa)_

There are generally three categories of people that interact with Edgex:

**Users**

Folks that use Edgex as-is and do not need or intend to make any modification/additions to the code base.


**Developer / Contributor**

These are people that want to run, modify and build the existing edgex codebase, and in some cases, contribute their modificiations back to the Edgex open sosurce effort.


**Hybrid**

This category of users often run the main edgex services as-is, however, they also add custom microservices that serve their business use case. These services interact with the rest of Edgex. This is the category I belong to (in the context of this project).

---

### About the Project
_[Disclaimer] I'm working on this project primarily for learning purpose. As a result, the product requirements might not exactly match what obtains in a production use case._

For this project, I fall into the category of EdgeX users known as Hybrid User (explain meaning and link to section of youtube video)
In this article, I would focus on setting up and integrating my device service with the reset of Edgex.
The plan is to build a custom [device service](https://docs.edgexfoundry.org/3.1/microservices/device/Ch-DeviceServices/) that interacts with the sensors and leverages on the other components of Edgex to process the data, and then expose the output via a custom [application service](https://docs.edgexfoundry.org/3.1/microservices/application/ApplicationServices/)


---


### Setting up Docker
First, I have to setup docker on my system. Edgex has an [option for using Ubuntu snaps](https://docs.edgexfoundry.org/3.1/getting-started/Ch-GettingStartedSnapUsers/) to pull the various microservices images. I tried this approach on my PIs, however, I found [the docker alternative](https://docs.edgexfoundry.org/3.1/getting-started/Ch-GettingStartedDockerUsers/) more familiar and opted for it instead. Moreso, my developement environment is MacOS, so, Docker.

I already have docker-desktop running on my machine, so I don't need [a fresh install](https://www.docker.com/products/docker-desktop/). Next thing I need to create my project's directory, `digital-elevator/`and then do [the git setup ritual](https://docs.github.com/en/repositories/creating-and-managing-repositories/quickstart-for-repositories).

Now that docker and git are sorted, let's go into the crux of the Edgex setup.

Edgex provides a series of pre-configured docker compose files for various microservice cmobinations. For the purpose of this project, I don't need an authentication layer, so I will be going with the `no_secty` variant. My edge device is a [Google Coral Dev Board](https://coral.ai/docs/dev-board/get-started), which runs an arm64 chip, so I'll pull the matching [compose file](https://github.com/edgexfoundry/edgex-compose/blob/main/docker-compose-no-secty-arm64.yml). I also pulled the [x86 version](https://github.com/edgexfoundry/edgex-compose/blob/main/docker-compose-no-secty.yml) because I am developing on an Intel Mac. I then rename these files to match my preference.

Inside my project rooot dir (`/digital-elevator`), I run the docker compose up for my dev environment, using the `-f` flag to specify the appropriate compose file. (Note a docker sign in is required, if you hadn't done so previously). Now I have Edgex running!


---


### Set up the Signals Service
`signals-service` is the name I gave my device service. This microservice would be responsible for interaction with the various [devices](https://docs.edgexfoundry.org/3.1/general/Definitions/#device) needed for my digital twin. It's main function is processing real-world signal input via sensors, and in some cases, communicating back to the actuators.

For this project, I need to be able to read and respond to the following signals from the elevator.
- Speed (via a 3-axis accelerometer)
- Orientation (via same 3-axis accelerometer)
- Audio (via a microphone)
- Temperature (via a temperature sensor)

Edgex Foundry provides a [template repo](https://github.com/edgexfoundry/device-sdk-go) that helps to get a device service up and running quickly, as well as [a very detailed walkthrough](https://docs.edgexfoundry.org/3.1/walk-through/Ch-WalkthroughDeviceService).
While I fully leveraged the template repo, I didn't quite follow the walkthrough to the later. This is because some assumptions were made which didn't quite apply to my scenario. Also, a number of the setup was done by making API calls to various components in Edgex. This approach seemed to imperative for me, so I took the declarative alternative instead - defining the necessary service components in yaml/json files. This approach feels a lot more natural. Thankfully edgex providsd example files (ableit in yaml format), that detail the configuration.


Firstly, I need to clone the [device-sdk-go](https://github.com/edgexfoundry/device-sdk-go) repo, after which the rest of the setup can take place.


Below, I itemise the steps (in sequence) to customize this template to my `signals-service`:
_*NOTE*: I will be attaching screenshots/links for code changes that I found not explicitly stated in the walkthrough, but are important._

##### 1. Copy the /example dir into my project dir (`digital-elevator`) and rename to `signals-service`.

---

##### 2. Copy the dockerfile from the cmd direcvory into the `signal-service` base dir (this is because I am using a docker setup to run my devuce service alongside the other edgex services).

---

##### 3. Copy `Makefile` and `version.go` from the `device-sdk-go/` dir to `signals-service/`.

---

##### 4. Move the content of the `cmd/device-simple/*` into `signals-service/core` and delete `cmd/device-simple`.

---

##### 5. Move the `attribution.txt` file to `signals-service/` (optional).

---

##### 6. Update `core/main.go` to replace the `/driver` import with that of my project.
{{< img src="images/core_main_driver.png" alt="core/main.go" caption="replace default driver import.">}}

---

##### 7. Update the reference to `/config` inside of `core/driver/` to use that of my project (`/signals-service/core/config`).

---

##### 8. Run `go mod init`, `go mod tidy` in the base dir of the service to set up this service dir for go module functionalies (e.g dependency tracking).

After these steps, this is what my directory structure looks like:
{{< img src="images/directory_structure.png" alt="direcory structure" caption="signals-service directory structure">}}

---

##### 9. Next, I update `signals-service/Dockerfile` to suit my project structure.
_NOTE: if you opted out of step 5 above, you'd need to remove the section that copies `attribution.txt` into the volume._
{{< img src="images/dockerfile_changes.png" alt="dockerfile" caption="Update signals-service/Dockerfile to match project">}}

---

##### 10. I adapt `signals-service/Makefile` to fit my project structure as well.
{{< img src="images/makefile_changes.png" alt="makefile" caption="Update signals-service/Makefile to match project">}}

---

##### 11. From `signals-service/`, I run `make build` to confirm that my project builds successfully after all these changes.
{{< img src="images/make_build_success.png" alt="make build success" caption="signals-service builds successfully!">}}

---

##### 12. The last bit of this set up is to get signals-service to work with the rest of Edgex, through the docker comopose. The screenshot below shows the changes that got this done:
{{< img src="images/docker_compose_inclusion.png" alt="docker compose" caption="include signals-service in docker-compose file." >}}

*A few notes about these changes:*
- I set the image name to match what I had defined my my makefile (where I build the image)
- I specify that the service needs to wait for some other edgex services (consul, core-data) to have started before it does, as it depends on them.
- I add the container to the same network shared by the other edgex services
- The port this container receives request on matches what I specified in the Dockerfile.

---

##### 13. Finally, I navigate up to the project's base dir(`digital-elevator/`) and re-run `docker compose`, so that the rest of Edgex can now recognize my `signals-service`.
{{< img src="images/project_setup_complete.png" alt="rebuild docker" caption="It works!" >}}



---



### In the next article for the series,
I would be sharing how to setup my first device and audio-sensor, and get it to communicate data read from the microphone sensor to the rest of Edgex, via the signals-service I just set up. Ths would include:

- Setting up the [device](https://docs.edgexfoundry.org/3.1/general/Definitions/#device) in Edgex.
- Preparing the physcial device for Edgex (recall that so far, I've been running Edgex in docker on my development machine).
- Deploying the digital-elevator containers on the device.


Thank you for reading, I do hope you found this article useful.
While I have not set added the comments feature on my blog, please reach out to me with any questions or thoughts on [Twitter](https://www.twitter.com/@osifo__).

I hope share the next part soon!

