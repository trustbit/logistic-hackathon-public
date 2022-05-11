<p align="center">
  <a href="https://www.bitmovin.com">
    <img alt="Trustbit Hackathon: Sustainable Logistics Simulation" src="images/header.jpeg" >
  </a>

  <h4 align="center">This is the central repository for participants of the <br><a href="https://trustbit.tech/hackathon" target="_blank">Trustbit Hackathon: Sustainable Logistics Simulation</a> which explains how to participate in the competition and links to all relevant resources like agent templates. The build status of participants agents can be seen in the <a href="actions">GitHub Action history</a> of this repository.</h4>

  <p align="center">
    <a href="LICENSE"><img src="https://img.shields.io/badge/License-MIT-yellow.svg" alt="License"></img></a>
        <a href="https://trustbit.tech"><img src="https://img.shields.io/badge/Organizer-Trustbit-%23006871" alt="Join Slack chat"></img></a>
    <a href="https://join.slack.com/t/trustbitsusta-vl26615/shared_invite/zt-17i36qlc1-h6L0GsJov2gPLLSYFaqNmw"><img src="https://img.shields.io/badge/Slack-join%20chat-green" alt="Join Slack chat"></img></a>

  </p>
</p>

## Overview
Experts in data science, machine learning and software development will work together to find efficient and sustainable logistics solutions for the EU at the Trustbit Hackathon: Sustainable Logistics Simulation on 13 May 2022.

Freight transports, delivery routes and logistics systems are key drivers of the European economy. They allow countries to work together across borders, achieve better results together and find efficient and sustainable solutions for the future through cooperation and collaboration. Trustbit aims to foster this cooperation and brings together experts, developers and scientists to demonstrate what we can achieve by joining forces.

You do not have to be an expert in machine learning or data science. If you know how to code in one of our supported languages (Python and Java) you are ready to go. A simple script making decisions via if / else branches might even be better than an involved machine learning model. We are excited to see what you can achieve!

## Simulation
The simulation is represented by a map of locations connected by a network of streets. Some locations are special, as they produce goods, which will need shipping to other locations. The participants of the competition will get to program trucks which can load the cargo and deliver it. The goal is to be more efficient than the trucks of competitors.

This is very much like economic games you might have played in high school.

### Runtime and rules

So how does the simulation work exactly? The simulation runtime is provided by Trustbit and is responsible for keeping track of all stats like the current list of cargo offers (= produced goods), the trucks on the map, the current simulation date and time, fuel cost and many more. 

When the simulation is started it loads the world map, generates an initial list of cargo offers and spawns the trucks of the participants randomly on the map. Then the simulation will iterate through the list of trucks and will ask each one of them for their next move decision.

### Trucks
Trucks can decide to deliver a cargo offer, drive to a specific location empty, or let the driver sleep for a specific amount of time:

- `DELIVER` [cargo uid] - The truck reserves a specific cargo offer, drives to the current cargo location, loads the cargo onto the truck, plans a route to the cargo destination, drives there and unloads the cargo. Depending on the distance and the current stats of the truck driver, it might be necessary to put in a rest along the way. The same applies if the truck arrives outside of the destinations business hours.
- `ROUTE` [location name] - A truck might decide to plan a route and drive to a specific location, although it is currently empty. For example a truck could anticipate, that a lucrative cargo offer will soon be available at a location and wants to minimize the pick up and delivery time.
- `SLEEP` [number of hours] - When there are no cargo offers available, or when a truck driver is already on duty for a long time, it might be the best strategy to rest for a few hours. 

Each action by a truck agent is an atomic action which will always succeed. After each decision is executed, the truck will be able to decide for the next move.

### Simulation time
The current time is represented by the float `time` in the simulation, which has a range of 0.0 to 23.99. A value of 13.25 means `13:15`. Each action of a truck takes time:

Driving a truck on the map, takes a certain amount of time, which is calculated by the distance and the speed limits of a route. The cargo offers cargo delivery takes 5 hours, the truck will execute all necessary steps. Additional time might pass, if the driver has to rest, due to resting time regulations (see next section).

Example: 
- `loc`=A, `time`=8 
  
  The truck decides at 8:00 to drive from A to B, which will take 5.5 hours.

- `loc`=B, `time`=13.5

  The truck arrives at 13:30 at location B.


### Resting time of truck drivers

The drivers are allowed to drive for a certain amount of time until they need to rest. The simulation will count the driving hours and make it available to the truck agent scripts via the `driving_non_stop` counter.

The following simplified regulations apply:
  - Reserving, loading and unloading cargo is not tracked by the `driving_non_stop` counter.
  - Each hour driven will increase the `driving_non_stop` counter of the truck driver by 1 hour.
  - Each hour sleeping will reduce the `driving_non_stop` counter of the truck driver by 1 hour.
  - Exception: should the driver's `driving_non_stop` counter reach 9 hours, they will be forced to rest for 11 hours, reducing the counter to 0 again.

Example: 
 - `loc`=X, `time`=12, `driving_non_stop`=0 
    
    The driver decides to pick up a cargo offer from origin A and deliver it to destination B. It will take them 5 hours to reach location A and will take another 4 hours to deliver the cargo to location B. So the truck drives to location A.
 
 - `loc`=A, `time`=17, `driving_non_stop`=5
 
    The truck arrives at location A and the cargo is loaded onto the truck. The truck starts the journey towards it's destination. 
    
  - `loc`=A->B, `time`=21, `driving_non_stop`=5
    
    After 4 hours, the driver will need to rest for 11 hours as the `driving_non_stop` counter reached 9 hours. The simulation will do that automatically.

  - `loc`=A->B, `time`=8, `driving_non_stop`=0
      
    The truck driver is rested now and drive another 2 hours to reach its destination.
    
  - `loc`=B, `time`=10, `driving_non_stop`=2
    
    At location B the cargo will be unloaded. Would the truck have arrived outside of the business hours of the destination, the driver would automatically sleep until the destination is open for business.

### Opening hours of destination locations


### CO2 emissions
Driving emits CO2 which is calculated via a simplified COPERT4 formula (see also http://emisia.com/content/copert-documentation). The simulation tracks the emissions for all trucks on the map. Try to keep your emissions as low as possible, while maximizing your profit.

### Cargo offers

## Agent template repositories and competition build system

**We recommend to perform the steps of this section at the very beginning of the hackathon, just by using one of the supplied language templates without any changes. After the agent build is setup you can start to improve the truck agents behavior.**

### 1. Create private repository from a template
To get you started quickly, we created several template repositories for you. Depending on your language preference, click on one of the links below and then click the `Use this template` button to create a new **private** repository in your GitHub account with all the contents of the template. Make sure to make this new repository private, as otherwise all of your competitors will be able to see your code.

- [Java agent template repository](https://github.com/trustbit/logistic-hackathon-agent-template-java)
- [Python agent template repository](https://github.com/trustbit/logistic-hackathon-agent-template-python)

</br>
<img src="images/use-as-template.jpeg" width="640"/>
</br>

### 2. Add team members as collaborators to your private repository

If you are participating in a team, just collect the GitHub handles of you team members and add them in the [Collaborators and teams](/settings/access) page of your repository's settings. That way all of them will be able to contribute code.

<img src="images/collaborators.jpeg" width="640"/>
</br>

### 3. Create a new SSH key for the competition build system
Now we need to make sure the competition build system has access to your private repository. We will do that by creating an SSH key pair and by adding the public key to your repository's `Deploy keys` and by sending the private SSH key to our support staff, who will add it to our build system.

Steps in detail:

- Execute the following command on your local machine:

    `ssh-keygen -t rsa -b 4096 -C "your_email@example.com" -q -N "" -f hackathon`

- This will create 2 files in your current directory: 
  - `hackathon.pub` - the public key
  - `hackathon` - the private key

- Go to the [Deploy keys](settings/keys) page of your repository's settings and create a new deploy key `Hackathon` there and paste in the contents of your **public key file**.

<img src="images/deploy-key.jpeg" width="480"/>

- You will then need to send the **private SSH key file** to us via Slack. The best way to do that is to go to our hackathon Slack workspace, create a new channel there `support-[unique-team-id]` and invite your team members and our support staff via Slack handle `@support-staff`. To reduce the likelihood of copy/paste errors, always upload your key as a file to Slack. If you did not join the Slack workspace yet, you can do that [here](https://join.slack.com/t/trustbitsusta-vl26615/shared_invite/zt-17i36qlc1-h6L0GsJov2gPLLSYFaqNmw).

- Add your teams details in the [agents.json](agents.json) file and open a new PR with those changes. The secret for your team will be created by our support staff in the meantime. The naming convention is `TEAM_[UNIQUE_ID]`. Here is a template for a valid entry in the `agents.json` file:

```json
    {
      "unique_id": "trustbit",
      "is_fleet": false,
      "github_repository": "trustbit/example-truck-agent",
      "github_private_ssh_key_secret": "TEAM_TRUSTBIT"
    }
```
- If you are participating alone set `is_fleet` to `false`. If you are participating as a team set it to `true`.

- As soon as your PR is merged, we will have your repository included in our build, which is scheduled to run **every 5 minutes** and which will provide new versions of your agent to the simulation. You can check on the status of your builds here: https://github.com/trustbit/logistic-hackathon-public/actions/workflows/docker-images.yml

## Competition runs

**Please follow the instructions under [Agent template repositories and competition build system](#agent-template-repositories-and-competition-build-system) first, to get the initial setup out of the way quickly.**

One simulation run takes **exactly 5 minutes**. Before each run we pull the latest versions of your truck agents. As the [build](https://github.com/trustbit/logistic-hackathon-public/actions/workflows/docker-images.yml) also runs every 5 minutes it can happen, that your newest version will miss a simulation run, but it will definitely be part of the next one then.

So what happens during a simulation run exactly? See the following step-by-step explanation:

1) Fetch the latest version of the [agents.json](https://github.com/trustbit/logistic-hackathon-public/blob/main/agents.json) file, so the simulation knows, which teams want to participate.
1) Start Docker containers for all the truck agents and make sure that they are reachable by the simulation runtime. If one of the truck agent containers is not reachable (maybe no successful build happened yet for a team, or the agent returns errors when called), it will be excluded from the simulation run.
1) We pre-warm all truck agents to make sure they can service simulation requests as quickly as possible.
1) Generate a list of available cargo offers and randomize the order of truck agents.
1) Loop through the list of truck agents and ask each one sequentially for a decision on their next move.
1) Execute the desired move. 
1) All truck agents are competing for the same list of offers. For example, if a truck agent decides to `DELIVER` a cargo offer, the same offer will not be available for the next truck agent.
1) Each move takes a certain amount of time, depending on factors like maximum speed on the roads and the distance a truck agent has to travel. The simulation keeps track of the truck agent actions and will call them again after their current move is finished.

TODO: continue here
1) Truck agents have a `balance` which represents the money in their bank account. Fuel prices and monthly

### Results of a simulation round
- Link to trace bucket
- Link to Grafana
- Explanation how to display traces
- Add image for Grafana

## We are hiring