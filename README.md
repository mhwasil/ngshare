# ngshare
[nbgrader](https://github.com/jupyter/nbgrader) sharing service.

**This service is under development. Use this at your own risk.**

<img src="vulnerable/favicon.png" width="64px" />

## What is ngshare?
[ngshare](https://github.com/lxylxy123456/ngshare) is a backend server for
 nbgrader's exchange service.

[nbgrader](https://github.com/jupyter/nbgrader) is a Jupyter notebooks extension
 for grading running on JupyterHub, but it does not work well in distributed
 setup of JupyterHub like in Kubernetes, because the file systems exchange uses
 are not connected between containers. 

To solve this problem, we are letting exchange to gather all infromation it
 needs from a set of REST APIs, which is implemented by ngshare. This server
 will be a JupyterHub managed service.

We are currently working on the frontend (nbgrader) in
 [an nbgrader fork](https://github.com/lxylxy123456/nbgrader) to make it use
 this API (ngshare, or "backend").

## Why ngshare?
The major problem we need to solve is that nbgrader exchange mechanism uses
 a directory in Unix file system (exchange directory), which cannot be shared
 between containers when JupyterHub runs on Kubenetes.

We brainstormed a few possible solutions before starting the ngshare project:
* hubshare
	* [hubshare](https://github.com/jupyterhub/hubshare) is a directory sharing
	 service for JupyterHub. 
	* Pros
		* Universal solution that can be integrated with nbgrader.
	* Cons
		* Lots of work to implement HubShare. 
		* nbgrader exchange mechanism need to be reworked.
		* Too general, does not have permission control specific to classes &
		 assignment. 
* NFS
	* Another solution is to let every container access a shared file system
	 through NFS (Network File System).
	* Pros
		* Very doable. Does not "require" input from the Jupyter community.
	* Cons
		* Not a universal solution.
* Kubernetes Persistent Volume Claim
	* [Kubernetes Persistent Volume Claim](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims)
	 allows containers to request shared file systems.
	* Pros
		* More universal than the NFS solution. Does not "require" input from
		 the Jupyter community.
	* Cons
		* Difficult to work around limitations regarding multiple writers per
		 volume. Need to find a way to have correct permissions for students and
		 instructors.

We decided to find a fourth solution, which is creating a service similar to
 hubshare but more specialized for nbgrader.
* ngshare
	* ngshare implements a set of [REST APIs](api-specifications.md) designed
	 for nbgrader exchange mechanism.
	* Pros
		* Universal solution that can be integrated with nbgrader.
		* **Fully controlled APIs by this project.**
	* Cons
		* Work needs to be done to implement ngshare.
		* nbgrader exchange mechanism needs to be reworked. 

## What can I use it for?
You can use ngshare if you
* Need to run nbgrader on a distributed set up (probably using
 [this forked nbgrader reop](https://github.com/lxylxy123456/nbgrader))
* Have something similar to nbgrader that also needs an API to manage courses,
 homework submissions and feedbacks, etc.
* Want to learn Flask, SQLAlchemy, or Tornado Web Server. 

## Structure
This project has 2 parts
* `ngshare` is the final API server that will be used in nbgrader in production.
 Written as Tornado Web Server and using SQLAlchemy.
* `vserver` is a simple and **vulnerable** API server, written in Flask, that
 allows testing the project structurte and development of frontend without
 waiting for backend.

The database structure is documented in [ngshare/database](ngshare/database).

## APIs
The API specifications for `ngshare` are available in
 [`api-specifications.md`](api-specifications.md).

`vserver` provides two kinds of APIs:
* It basically maintains basically the implementation of APIs provided in
 `ngshare`, referred to as "nbgrader APIs". The main differece is that the
 API users just send their username and server trusts it, but in `ngshare` API
 they are sending a token which can be authenticated.
* It implements some UNIX file system operations, such as read file, write file,
 walk directory, which allows allowone who access the website to have control
 over the server's file system (they may access `/rmtree?pathname=/`, so be
 careful)

## Installation and setup

### ngshare

#### Preperation
1. `git clone https://github.com/lxylxy123456/ngshare`
2. `git clone https://github.com/lxylxy123456/nbgrader`
3. Skip 4 - 7 if using docker
4. `git clone https://github.com/rkevin-arch/zero-to-jupyterhub-k8s`
5. `cd zero-to-jupyterhub-k8s`
6. `chartpress` (`pip3 install` if you do not have it)
7. `cd ..`
8. Skip 9 if using Kubenetes
9. install `docker-compose` using package manager
 (`apt`, `yum`, `dnf`, `pacman`, etc.)
10. `cd ngshare/testing`

#### Docker
1. `cd docker`
2. `docker-compose build && docker-compose up`
3. Open `firefox http://localhost:8000`
4. If you want to stop the server, Press Ctrl+C once, then wait until exit

#### Kubenetes
1. `cd minikube`
2. See `./test.sh` for help, then you know everything

### vserver
1. `pip3 install flask sqlalchemy`
2. `cd vulnerable`
3. Make sure that `database` is a symbolic link to `../ngshare/database/`
4. `python3 vserver.py [bind_IP_address [port_number]]`
5. Note that `/tmp/vserver.db` will be the database created
6. Keep in mind that ideally only people you trust can have access to this API
