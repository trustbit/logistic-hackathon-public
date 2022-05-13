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

You do not have to be an expert in machine learning or data science. If you know how to code in one of our supported languages (C#, Java, Python) you are ready to go. A simple script making decisions via if / else branches might even be better than an involved machine learning model. We are excited to see what you can achieve!

## Simulation

We simulate a road network in Europe. For the purpose of the hackathon, it is simplified.

![image-20220512095546609](images/image-20220512095546609.png)

The world map for this road network is static and available: [map.json](data/map.json). **Map schema** is explained later.

The world map is represented by locations connected by a network of roads.  Some locations are special, as they produce goods, which will need shipping to other locations. 

Locations for producing and consuming goods are semi-deterministic and are not revealed publicly. You can analyse the **simulation trace data** to find out the patterns.

The participants of the competition will get to program trucks which can load the cargo and deliver it.  **The goal is to make money by delivering the cargo**. 

If you run out of money, your truck(s) are suspended for the current simulation run.

This is very much like economic games you might have played in high school.

### Implementation

So how does the simulation work exactly? 

The simulation runtime is provided by Trustbit and is responsible for keeping track of all stats like the current list of cargo offers (= produced goods), the trucks on the map, the current simulation date and time, fuel cost and many more. 

When the simulation is started it loads the world map, generates an initial list of cargo offers and spawns the trucks of the participants randomly on the map. Then the simulation will iterate through the list of trucks and will ask each one of them for their next move decision.

Afterwards, we simulate the passing of time via [discrete-event simulation](https://en.wikipedia.org/wiki/Discrete-event_simulation). Trucks travel, time passes and we jump forward to the next important event.

### Your Workflow

1. You code an agent that controls truck(s) and submit it into the competition.
2. That agents gets packaged as a docker container
3. Regularly (every 5-10 minutes) we pull all agents in and start a new simulation run.
4. You can observe the real-time dashboards during the run. 
5. You can also download full simulation trace immediately after the run ends

The goal is to **observe, iterate and improve**.

We have two sets of rules:

- **Efficiency rules** - in effect from the start of the competition
- **Sustainability rules** - come in effect in the second half of the Hackathon

### Efficiency Rules

 These rules are in effect since the start of the game.

- Your **truck gets money for delivering** cargo. Prices are market-driven. Certain cargo types tend to have better margins out-of-the-box.
- You spend money on gas (fuel consumption uses COPERT formula for a loaded/empty truck and diesel price of `2.023`)
- **Every week, you pay fixed operational costs**. So standing idle and doing nothing is a loosing strategy. You need to hustle to keep.
- If your balance is negative at the end of the week, the truck is bankrupt until the end of the simulation run.
- Weekly revenue is taxed progressively.

### Sustainability Rules

These rules come into effect for the second part of the hackathon.

- **CO2 emissions** (computed via COPER) have an additional associated cost (CO2 offset cost)
- **Truck drivers get fatigued** over the time. Fatigue accumulates and increases the chance of road incidents. Incidents cause delay and trigger an immediate rest. Full 8-hour sleep is needed to eliminate fatigue.
- **Locations have working hours**. If a truck arrives at a location outside of its working hours, it has to wait.

### Trucks

You control trucks by implementing a `decide` function. This function is called whenever the simulation doesn't have a plan for the truck (e.g. at the start of the simulation).

The simulation runtime will build a current world state for the truck (list of available cargo) and request a decision.

Trucks can decide to **deliver a cargo**, **drive to a specific location** empty, or **let the driver sleep** for a specific amount of time:

- `DELIVER CARGO_UID` (e.g. `DELIVER 23`) - The truck reserves a specific cargo offer, drives to the current cargo location, loads the cargo onto the truck, plans a route to the cargo destination, drives there and unloads the cargo. Fatigue and working hour effects can apply here, if _Sustainability Rules_ are in effect.
- `ROUTE LOCATION` (e.g. `ROUTE Berlin`) - A truck might decide to plan a route and drive to a specific location, although it is currently empty. For example a truck could anticipate, that a lucrative cargo offer will soon be available at a location and wants to minimize the pick up and delivery time.
- `SLEEP HOURS` (e.g. `SLEEP 1` or `SLEEP 8`) - When there are no cargo offers available, or when a truck driver is already on duty for a long time, it might be the best strategy to rest for a few hours. Note, that sleeping 8 times for one hour doesn't constitute a full rest.

Each action by a truck agent is an atomic action which will always succeed. After the decision is fully executed, the truck will be able to decide again.

### Simulation time
The current time is represented by the float `time` in the simulation. It represents the number of hours that have passed since the start of the simulation. 

- A value of 13.25 means `13:15`. 
- `26.0` means `1 day 2 hours`

Each action of a truck takes time. Driving a truck on the map, takes a certain amount of time, which is calculated by the distance and the speed limits of a route. The cargo offers delivery takes 5 hours, the truck will execute all necessary steps. Additional time might pass when _Sustainability Rules_ are active, and the driver gets into an accident due to insufficient resting time (see next section).

Example: 
- `loc`=A, `time`=8 
  
  The truck decides at 8:00 to drive from A to B, which will take 5.5 hours.

- `loc`=B, `time`=13.5

  The truck arrives at 13:30 at location B.

Hint: you can get time of the day by doing a modulo operation: `tod = time mod 24`


### Resting time of truck drivers (Sustainability Only)

When _Sustainability Rules_ are active, driver fatigue mechanics come into play. 

Drivers can get tired over time. More time has passed since the last full rest (8 hours of undisturbed SLEEP), the higher the chance of road incidents during the journey. 

Road incidents cause a delay and trigger an immediate `SLEEP 8` afterwards.

### Working Hours (Sustainability Only)

When _Sustainability Rules_ are in active, cargo delivery locations have working hours. If a truck arrives at such a location outside of the working hours, it will have to wait until it is allowed to unload its cargo.

If waiting time is 8 hours or more, it counts as a full rest.


### CO2 emissions (Sustainability Only)
Driving emits CO2 which is calculated via a simplified COPERT4 formula (see also http://emisia.com/content/copert-documentation). The simulation tracks the emissions for all trucks on the map. Try to keep your emissions as low as possible, while maximizing your profit.

CO2 emissions are counted as expensses, based on the CO2 offset cost per kg.

### Cargo offers

Simulation runtime tracks cargos available for delivery. Whenever it is time for a truck to make a decision, it will receive a personalized list of cargo offers. 

```json
{
  "uid": 100,                # unique cargo ID
  "origin": "Zaragoza",      # city to pick it up
  "dest": "Innsbruck",       # destination
  "name": "Fruits",          # kind
  "price": 1778.0,           # money you get for delivery
  "eta_to_cargo": 8.16,      # ETA from current loc to pickup loc
  "km_to_cargo": 780.0,      # kms to pickup location
  "eta_to_deliver": 27.76,   # ETA for picking up and delivering
  "km_to_deliver": 2730.0    # total kms to pick up AND deliver
}
```



ETA and KM estimates are relative to the current position of the truck. They account only for the driving time, but don't account for the loading/unloading and waiting times.

You can check out for the sample cargo offers in [data/](data/) directory.

## Available Data

We have a set of data available for the deeper analysis:

- World map (static)
- Simulation traces
- Observability Dashboard

### World Map

You can [download](data/map.json) the world map. It describes the road graph as a collection of locations and roads that connect them. Estimated driving speed is also provided.

### Simulation Traces

Simulation traces could be downloaded from the [traces folder](https://traces.endpoints.trustbit-hackathon.cloud.goog). These are JSON files stored in [Chrome Tracing format](https://docs.google.com/document/d/1CvAClvFfyA5R-PhYUmn5OOQtYMH4h6I0nSsKchNAySU/preview).

Note: Simulation traces that were published before the competition start - demonstrate the mechanics, but use a slightly different world. Keep that in mind, if you decide to analyse in advance.

You can analyse the traces manually or load into the `chrome://tracing` tool. 

Navigate to `chrome://tracing` in Chrome or Chromium browser, then drag-and-drop a trace file into the tracing UI.

There you can see a detailed breakdown of every single action that happened in the simulation, for all the participants and NPC agents.

![image-20220512105849865](images/image-20220512105849865.png)

You can zoom, filter and dig into the details. Events tend to have associated data:

![image-20220512110108793](images/image-20220512110108793.png)



#### Caveat - ms instead of hours

Chrome tracing simulator isn't suitable for displaying large intervals like hours. So we "hack" and pass time and durations as milliseconds instead. When you see a millisecond unit, convert it mentally into hours.

E.g.:

- "Start 3,839.524 ms " means "Start at 3839.524 hours since the start of the simulation"
- "Wall duration 2.034 ms" means "Wall duration of 2.034 hours" (roughly two hours and a minute)



#### Manual Analysis

The file looks like the snippet below. Actions of all teams are intertwined there:

```json
{"name": "DRIVE", "ph": "B", "ts": 18941.0, "pid": 5, "tid": 1, "args": {"a": "Salzburg", "b": "Linz", "fuel_l": 24.19, "co2_kg": 63.86, "cargo": 15, "km": 107, "delay": 0, "incident_cost": 0}},
{"name": "DRIVE", "ph": "E", "ts": 20156.90909090909, "pid": 5, "tid": 1},
{"pid": 5, "ts": 20156.90909090909, "ph": "C", "name": "balance", "args": {"value": 24806}},
{"name": "DECIDE", "ph": "i", "ts": 19000.0, "pid": 8, "tid": 0, "s": "t", "args": {"cmd": "SLEEP", "arg": "1"}},
{"name": "SLEEP", "ph": "B", "ts": 19000.0, "pid": 8, "tid": 0},
{"name": "SLEEP", "ph": "E", "ts": 20000.0, "pid": 8, "tid": 0},
{"pid": 8, "ts": 20000.0, "ph": "C", "name": "balance", "args": {"value": 8713}},
{"name": "DECIDE", "ph": "i", "ts": 19080.0, "pid": 4, "tid": 2, "s": "t", "args": {"cmd": "SLEEP", "arg": "1"}},
{"name": "SLEEP", "ph": "B", "ts": 19080.0, "pid": 4, "tid": 2},
{"name": "SLEEP", "ph": "E", "ts": 20080.0, "pid": 4, "tid": 2},
```

Unique identifiers are `pid` (team ID). You can look them up for your truck at the beginning of the file in the `M` (Metadata) records:

```json
{"name": "process_name", "ph": "M", "pid": 1, "args": {"name": "npc_truck_naive"}},
{"name": "thread_name", "ph": "M", "pid": 1, "tid": 0, "args": {"name": "npc_truck_naive/0"}},
{"name": "process_name", "ph": "M", "pid": 2, "args": {"name": "npc_truck_rinat"}},
{"name": "thread_name", "ph": "M", "pid": 2, "tid": 0, "args": {"name": "npc_truck_rinat/0"}},
{"name": "process_name", "ph": "M", "pid": 3, "args": {"name": "npc_truck_seb"}},
{"name": "thread_name", "ph": "M", "pid": 3, "tid": 0, "args": {"name": "npc_truck_seb/0"}},
{"name": "process_name", "ph": "M", "pid": 4, "args": {"name": "npc_fleet_profit_seeking"}}
```

If you have multple trucks, they will share the same `pid` but will have different `tid`.

### Observability Dashboard

We have  dashboards with real-time insight into the system across the simulation runs. 

Each run takes 5-10 minutes, and you can see it as a separate peak on the charts.

- [General dashboard](https://telemetry.endpoints.trustbit-hackathon.cloud.goog)

![image-20220512110929530](images/image-20220512110929530.png) 

## Agent template repositories and competition build system

**We recommend to perform the steps of this section at the very beginning of the hackathon, just by using one of the supplied language templates without any changes. After the agent build is setup you can start to improve the truck agents behavior.**

### 1. Create private repository from a template
To get you started quickly, we created several template repositories for you. Depending on your language preference, click on one of the links below and then click the `Use this template` button to create a new **private** repository in your GitHub account with all the contents of the template. Make sure to make this new repository private, as otherwise all of your competitors will be able to see your code.

- [C# agent template repository](https://github.com/trustbit/logistic-hackathon-agent-template-csharp)
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

- Fork this repository, add your teams details in the [agents.json](agents.json) file and open a new PR with those changes. The secret for your team will be created by our support staff in the meantime. The naming convention is `TEAM_[UNIQUE_ID]`. Here is a template for a valid entry in the `agents.json` file:

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

## We are hiring
We — a team of more than 30 people from over 10 nations — organise our daily work around trust and flexibility. You decide whether you want to work full-time or part-time. We are flexible and can adapt to your individual needs. We know that really good projects only happen if employees can concentrate on their personal strengths and deepen their interests. That’s why self-directed, continuous training is a central part of our daily work. To make this possible, we offer numerous opportunities to discover new things and learn from each other.

What’s it like to work as a Software Engineer, Designer, Project Manager, Data Scientist or Consultant at Trustbit? That’s exactly what we asked some of our colleagues, [enjoy watching](https://trustbit.tech/careers)! 🍿

<br/>
<a href="https://trustbit.tech/careers">
  <img src="images/trustbit.png" width="240"/>
</a>