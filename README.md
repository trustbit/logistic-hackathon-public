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


### Agent template repositories
Explanation that public repos are also supported but that competitors can see the code then
- logistic-hackathon-agent-template-java
- logistic-hackathon-agent-template-python

### Local simulation
- Instructions how to run simulation locally

### Competition simulation
- Link to Slack space
- Link to trace bucket
- Link to Grafana
- Explanation for agents.json
- Instructions how to get agents into the competition simulation
- Explanation how to display traces
- Explanation how and when the competition simulation is running


We launch that as a server in a cloud. Attach a few Grafana dashboards. Competitors could control trucks (individual or fleets) by uploading scripts that make decisions (use JS VM or something else that we can easily and safely sandbox). 





## We are hiring