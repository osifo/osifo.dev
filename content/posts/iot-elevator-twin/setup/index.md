---
title: "Building an Elevator Twin - Setup"
date: 2024-01-19T00:39:59+01:00
draft: true # Set 'false' to publish
tableOfContents: false # Enable/disable Table of Contents
description: 'In this series of artiicles, I share my learnings as I build a digital elevator IoT sytem using EdgexFoundry.'
categories:
  - iot
  - edgex
tags:
  - iot
  - edgex
---

In this multi-part series I'd be sharing about how I built a digital twin for elevators ussing open source technologies.

The IoT framework I'd be using for this project is Edgex Foundry.
A little summary about EdgexFoundry (Edgex)...
- Language
- Apache
- Jim Whhite IotTech
- Documentation (github, youtube)

Let's get started with my project.
For this project, I fall into the category of EdgeX users known as Hybrid User (explain meaning and link to section of youtube video)

Spinning up my environment
- setup docker on my system
- signin to docker
-  create a dir for my project (digital-elevator)
- go to the edgex/edgex-compose repo and copy the compose file that suits my use case.
- create a compose.yml file in my project base dir and paste the copied content into it.
- Inside digital-elevator, run docker compose up (--build -d) to confirm that environment setup works. Also check the services via their ports (4000 for edgex-ui and 8500 for consul)

Set up up a device-service
A device service is a ...(complete definition) for reading signal data via sensors (and in some cases, communicating back to the sensors). Add reference to the other EdgeX service layers/components.

In my case, I want to be able to read and respond to the following signals from the elevator.
(list signals here)

- clone the device-sdk-go repo from github and copy the /example dir into my project dir and rename to signals-service.
- copy the dockerfile from the cmd diir into the service's base dir (this is because i'm using a docker setup to run my custom service alongside the other edgex services)
- copy Makefile and version.go from the device-sdk-go/ dir to signals-service/
- moved the content of the cmd/device-simple/* into my signals-service/core and delete cmd/device-simple (i also moved the attribution.txt filie to my service'ss base dir)
- update core/main.go to replace the /driver import with that of my project
- update the reference to /config inside of core/driver/ to use that of my project (/signals-service/core/config)
- run go mod init, go mod tidy in the base dir of the service
- after this, my repo should look like this: (add screenshot)
- update my dockerfile in my service base dir to match my dir structure
- update the Makefile to suit my project structure as well
- from inside my signals-service, I run "make build" to be sure that my project builds successfully after the changes.
- next i need to update my project's docker compose file, so that my signals service can be managed by docker, alongside the other edgex services

