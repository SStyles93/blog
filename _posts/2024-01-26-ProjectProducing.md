---
title: Project Producing - First experience as a Producer
categories: [Producing]
image: /assets/images/projectproducing/PP_Intro.gif
---

Organizing the teams work, preparing meetings, resolving conflicts, all the tasks relative to game production in a team composed of 30 people will be exposed in this blog.

## Contextualization

During the last year of studies at the SAE-Institute the students of the Games Programming section have to create a game in collaboration with the Game Art and Audio Engineering sections. The purpose of the module is to simulate what is, for some, a first work experience in a professional-like environment.
One of the roles I inherited was to be the producer for both project. All the process, tools used and personal thoughts regarding this role will be exposed and commented throughout this article.

## Overview

At the beginning, all the students were teamed up and had to present 8 game pitches to the stakeholders. From these 8 pitches 4 were selected to be improved and presented again. In the end two projects were selected to be produced. For both projects the students had to rework the pitches, create a game design document and a technical document and present them at the beginning of June 2023.

![]({{ site.baseurl }}/assets/images/projectproducing/GK_Pitch.PNG "Pitch Girl and Kitty "){: width="100%"} _GK Pitch front page_ | ![]({{ site.baseurl }}/assets/images/projectproducing/VR_Pitch.PNG "Pitch project VR"){: width="100%"} _VR Pitch front page_ 

Once the pitches were presented, two game art students (one per project) were designated to create the concept art that would later be used to guide the artistic production. The concept art references and work was finished and presented for the beginning of July.

![]({{ site.baseurl }}/assets/images/projectproducing/GK_Concept.PNG "Concept Girl and Kitty"){: width="100%"} _Concept Girl and Kitty_ | ![]({{ site.baseurl }}/assets/images/projectproducing/VR_Concept.PNG "Concept project VR"){: width="100%"} _Concept project VR_

The game design was defined and improved in parallel by the game programmers. Two prototypes were created in parallel.

In september 2023 the head of game programming, [Elias Farhan](https://www.linkedin.com/in/elias-farhan-664373b5/), organized a master class with the CEO of [Old Skull Games](https://oldskullgames.com/), [Nicolas Brière](https://www.linkedin.com/in/nicolasbriere/), given by [William Marié](https://www.linkedin.com/in/william-mari%C3%A9-88ba4112/), a senior executive producer that gave us all the tools to achieve our projects with the best setup possible.

From that point and thanks to them, we had all the information necessary to elaborate a project planning from pre-production to release.

## The teams
The first action taken after the master class was to organize the teams and a global hierarchy for the projects. The project united the Game Programming, Game Art and Audio Engineering sections. In total we were 31 without accounting the external people that gave us master classes.  
The organization chart is the following

![]({{ site.baseurl }}/assets/images/projectproducing/Prod_Teams.PNG "Editor render"){: width="100%"}

The hierarchy was layed out from left to right. The stakeholders to whom we were accountable are placed on the left. Since I was producer, I was responsible for presenting the work done by the teams to them. Then, for each project, a product owner was nominated per project to follow the work and define the tasks according to the general objectives previously defined. The programmers were dispatched on the two projects according to necessities. The game artists were assigned to one project or the other. For the audio engineering section, we had two leads that would organize and dispatch work for both projects.


## Production Tools
This part of the blog is and overview and brief exposition of the tools we used for the game's production, it is categorized by domain.
### Programming tools
#### Unreal Engine 5.3
The game engine we had to use was [Unreal Engine 5.3](https://www.unrealengine.com/en-US/blog/unreal-engine-5-3-is-now-available). The obligation came from Elias Farhan, the head of game programming. Since Unreal is vastly used in the game industry it probably was a wise decision.
![]({{ site.baseurl }}/assets/images/projectproducing/UE5.PNG "UnrealEngine5.3"){: width="100%"}
_Unreal Engine 5.3 environment_

#### Perforce
Initially we wanted to setup [Perforce](https://www.perforce.com/) for both projects ot be able to work together, have version control and be able to lock file and avoid having a destructive workflow or too many merge conflicts.

![]({{ site.baseurl }}/assets/images/projectproducing/Perforce.PNG "Perforce environment"){: width="100%"}
_Perforce environment_

The problem was that, due to our internet provider, we couldn't set it up and finally installed [GitLab](#gitlab) on a self-hosted server.

#### GitLab

The self-hosting of [GitLab](https://about.gitlab.com/) was done by [Fabian Huber](https://www.fabianhbr.ch/) and the process is explained in one of his blogs that you can find [here](https://blog.stowy.ch/posts/devops-specialisation-projects/).
![]({{ site.baseurl }}/assets/images/projectproducing/Gitlab.PNG "Gitlab server"){: width="100%"}
_Gitlab server_

#### Nextcould
[Nextcloud](https://nextcloud.com/) was used for the team to transfer files, specifically the artists' assets. Once again the server was self-hosted and setup by Fabian. The details are explained [here](https://blog.stowy.ch/posts/devops-specialisation-projects/#nextcloud).
![]({{ site.baseurl }}/assets/images/projectproducing/Nextcloud.PNG "Nextcloud server"){: width="100%"}
_Nextcloud interface_


#### NAS
Since the project was self-hosted, we decided to create a secondary server to backup all the project.
To do so, I setup, with Fabian's help, a [NAS](https://en.wikipedia.org/wiki/Network-attached_storage) using a [Raspberry Pi 4 model B](https://www.raspberrypi.com/products/raspberry-pi-4-model-b/) with 4GB of [RAM](https://en.wikipedia.org/wiki/Random-access_memory)
and a WesternDigital [My Book Duo](https://www.westerndigital.com/products/external-drives/wd-my-book-duo-usb-3-1-hdd?sku=WDBFBE0160JBK-NESN).
The framework I used was [OpenMediaVault](https://www.openmediavault.org/) and the server was accessed by [Styles Studio](https://styles-studio.com/)' s domain name. I also added the [SSH](https://en.wikipedia.org/wiki/Secure_Shell) and [SSL](https://en.wikipedia.org/wiki/Transport_Layer_Security#SSL_1.0,_2.0,_and_3.0) protocols to access the server in a more secure way.
![]({{ site.baseurl }}/assets/images/projectproducing/NAS.PNG "NAS tools"){: width="100%"}

The last tool used regarding this server was [WinSCP](https://winscp.net/eng/index.php) to manage files in a more user-friendly way.
I also had to create the accounts and everything related to user identification.

Then, Fabian created a script to compress and send files to the NAS with a [cronjob](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/). The details of his implementation are explained [here](https://blog.stowy.ch/posts/devops-specialisation-projects/#backups).

### Audio tools
#### Pro Tools
For the production of sounds, the audio section principally used [Pro tools](https://www.avid.com/pro-tools) and divers sound banks or personal recordings.
#### Wwise
To facilitate the integration to UnrealEngine they used [Wwise](https://www.audiokinetic.com/en/products/wwise/) and, since we couldn't have perforce, transmitted their work to us with a USB key.

![]({{ site.baseurl }}/assets/images/projectproducing/Wwise.PNG "Wwise environment"){: width="100%"}
_Wwise environment_

### Artistic tools
#### 3DS Max & Maya
The tools used by the artists where principally [3DS Max](https://www.autodesk.com/products/3ds-max/overview?term=1-YEAR&tab=subscription) and [Maya](https://www.autodesk.com/products/maya/overview?term=1-YEAR&tab=subscription) for [Hard Surface modeling](https://www.3ds.com/store/cad/organic-modeling).

![]({{ site.baseurl }}/assets/images/projectproducing/3dsmax.PNG "3DS Max environment"){: width="100%"} _3DS Max environment_ | ![]({{ site.baseurl }}/assets/images/projectproducing/Maya.PNG "Maya environment"){: width="100%"} _Maya environment_

#### ZBrush
In some cases [Zbrush](https://www.maxon.net/en/zbrush) was used for organic modeling or sculpting.

![]({{ site.baseurl }}/assets/images/projectproducing/Zbrush.PNG "Zbrush environment"){: width="100%"}
_Zbrush environment_

Some examples of its use was for one of the characters in the project Girl and Kitty

![]({{ site.baseurl }}/assets/images/projectproducing/Girl.PNG "Girl Wireframe"){: width="100%"} _Girl Wireframe_ | ![]({{ site.baseurl }}/assets/images/projectproducing/Girl2.PNG "Girl Mesh"){: width="100%"} _Girl Mesh_

or for one of the humanoid assets the project VR.

![]({{ site.baseurl }}/assets/images/projectproducing/Cadaver.PNG "Cadaver character"){: width="100%"} _VR cadaver character_


#### Substance Painter & Designer
For the project VR, the artists used [Substance Painter](https://www.adobe.com/products/substance3d-painter.html) and [Substance Designer](https://www.adobe.com/products/substance3d-designer.html) to create the textures of their assets.

![]({{ site.baseurl }}/assets/images/projectproducing/SubstancePainter.PNG "Substance Painter environment"){: width="100%"} _Substance Painter environment_ | ![]({{ site.baseurl }}/assets/images/projectproducing/SubstanceDesigner.PNG "Substance Designer environment"){: width="100%"}  _SubstanceDesigner environment_

For the project Girl & Kitty, the texturing was done in Unreal Engine 5, for more information about that process you can read the blog post made about it [here](https://sstyles93.github.io/blog/posts/UnrealStylizedShaders/#contextualization)

## Organization Tools
This part is and overview and exposition of the tools we used for the project's organization.
### Google Drive
#### Weekly
One of the utilities of having a common drive was to store google slides. Since every week I had to present the general improvements done on the project, it was useful to be able to store our slides in a common folder. In general the product owners or myself were responsible for their creation.

![]({{ site.baseurl }}/assets/images/projectproducing/Drive_weekly.PNG "Weekly presentation"){: width="100%"} _Weekly presentation_

To keep everything as clear a possible our work was always exposed with the same hierarchy: General Production, Project Girl and Kitty and then the Project VR. With screenshots and bullet points for every member of the projects.

#### Project folders
Each project had its own folder with diverse documents inside. Going from technical documents to game design, these folders contained all the necessary resources created or found by the members.

![]({{ site.baseurl }}/assets/images/projectproducing/VR_Folder.PNG "VR Folder and content"){: width="100%"} _VR Folder and content_ | ![]({{ site.baseurl }}/assets/images/projectproducing/GK_Folder.PNG "GK Folder and content"){: width="100%"} _GK Folder and content_

#### Example Folder & Archives 
These folder contained all the documents received from William Marié to help us organize the production, pitch examples we used as references and all documents that were outdated.

![]({{ site.baseurl }}/assets/images/projectproducing/Example_Folder.PNG "Example Folder"){: width="100%"} _Example Folder_ | ![]({{ site.baseurl }}/assets/images/projectproducing/Archives_Folder.PNG "Archives Folder"){: width="100%"} _Archives Folder_

#### Admin folder
This folder was probably the most used by the product owners and myself. It contained all the documents related to the two projects and their

![]({{ site.baseurl }}/assets/images/projectproducing/Admin_Folder.PNG "Admin Folder"){: width="100%"} _Admin Folder_

### Trello

[Trello](https://trello.com/en)

![]({{ site.baseurl }}/assets/images/projectproducing/Trello_general.PNG "Trello workspaces"){: width="100%"} _Trello workspaces_

![]({{ site.baseurl }}/assets/images/projectproducing/Trello_board.PNG "Trello G&K board"){: width="100%"} _Trello Girl and Kitty board_

### Miro

[Miro](https://miro.com/)

![]({{ site.baseurl }}/assets/images/projectproducing/Miro_workspaces.PNG "Miro workspaces"){: width="100%"} _Miro workspaces_

### Discord

![]({{ site.baseurl }}/assets/images/projectproducing/Discord.PNG "Projects Discord"){: width="100%"} _Projects Discord_

## The process

What happened and why

## Management Style

Too nice ? too hard ? 
-> Why I couldn't be effective

## Personal Thoughts

- It was hard. 
- It took time
- It is a job that mustn't be neglected
- Can't be shared with 4 other jobs...

## Conclusion

- Nice experience
- Doesn't work in a school (colleagues don't respect planning & hierarchy)
