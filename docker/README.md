# Docker Templates

This folder contains my docker templates. The templates are really close to real life compose files in production on my servers. I wanted to propose something as close as possible as real life just without sensitive information because I find this more interesting and useful how people are really doing on their environment. This choice implies that the templates have some specificities, like labels for the use of Traefik or networks for my network segregation, that will most likely not fit your configuration. Take these templates as an inspiration for your set up. 
To help the potential geeky fellow who is reading this (I salute you), an explanation of my architecture is following.

## Architecture breakdown

### Folder structure
Each application is a folder. Each folder contains at least a compose-<NameOfTheApplication>.yml file and a env--<NameOfTheApplication>.env file, it can also contains some additional files like configuration files as in Traefik.
If a folder name starts with *network*, the compose file is meant to define explictly a network with a simple whoami container.

### Environments variables and files


### Network structure
The docker 

### Traefik Labels

### Watch Tower Labels
Some choices in the templates are dictated by my folder structure and deployement choice. 


## Potential Improvements
- Explore using *docker network create* instead of dumb container
- Move to Traefik 3
- Explore a nicer way to perform HTTP->HTTPS redirection with Traefik 3
- Add Healthstate for each container
- Add ressource constraints
