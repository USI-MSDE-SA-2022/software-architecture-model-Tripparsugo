## 1

Data collectors are one of the key components of our application: they retrive
the raw data we need and ingest it. Each of them uses a source of information
and converts the data to the mapping of we use for that type of data.
Some data might come from an external API while other from ```json```
 other types of file, the location of which could be local or remote.
Given we might potentially use tens or even hundreds of different sources a tool
to streamline the process of writing a collector, testing it and monitoring or
tweaking (ex: change activation timer on a collector) its function could prove to be useful.

 We want to implement our own DSL to write a collector that is transpiled to
 our implementation language. Programmers would still have the possibility to
 write a collector manually. In practice we could simply have a directory containing
 the collectors in our ```.ourdsl``` format and another where they get compiled
 into ```.ourlang``` files.

 Together with information about the collectors it would be usueful to have a platform
 to monitor the state of our application.


Some of the active components of our application write to our database and we could
query the data written on it as a sort of pseudo-heartbeat monitoring however this
would not be an effective strategy as:

* The issue might be with the data source and not with our component, we need to
distinguish the different cases


* We might want to test how much time some of our components take to elaborate
data and store this information over time. Perhaps a collector fetching from a
particularly poor API isn't able to provide data consinstently which we might know
about

Since we want to keep our components to keep operating alone we opt to turn them
into passive components but add a manager on top of them which runs them at specified
intervals of time or on request. In order to send these requests we add an API to manage
and monitor the component.


Logical view:

```puml

@startuml
skinparam componentStyle rectangle

!include <tupadr3/font-awesome/database>

title Logical View


component "Mobile App" as APP {

component "User Interface (React Native)" as UI
component "Redux Store (Redux + Thunk)" as RES
component "Redux Actions" as REA



component "LS Service (Realm)" as MSRV


component "Local Storage" as LS

UI -> REA
REA --> RES
REA -> MSRV
MSRV -> LS
}



component "Skip API service" as SA  {

component "Load Balancer" as LB1
component "Directory" as DIR1





component "Skip API (Kotlin + Spring framework)" as SA_{
component "Controller" as CTRL
component "Service" as SRV



component "User Repositories" as UREP
component "Cache db (Redis)" as CDB
CTRL -> SRV
SRV --> CDB
SRV -> UREP
LB1 -> SA_
LB1 -> DIR1
}



}






component "Data Aggregator API" as AGG{

component "Controller" as ACTRL
component "Manager" as AMAN
component "Data Aggregato" as AGG_


ACTRL --> AMAN
AMAN --> AGG_


}


component "Data Collector API" as COL{

component "Controller" as CCTRL
component "Manager" as CMAN
component "Transpiler" as TRS
component "Directory of Collectors" as CDIR
component "Collector 1" as COL1

component "Collector 2" as COL2

CMAN --> COL1
CMAN --> COL2
CCTRL --> CMAN
CMAN --> TRS

}



component "Dev API" as DAPI{
component "Controller" as DCTRL
component "Monitor" as DM
component "Storage" as DS

DCTRL -->  DM
DCTRL -->  DS

}

component "Watchdogs" as WS {
component "Watchdog 1" as W1
component "Watchdog 2" as W2

W1 -> W2 : monitors
W2 -> W1 : monitors



}

component "Services collection to save arrows on this uml" as SRVS {
component "Info storage service"
component "User storage service"
component "Data Collector API"
component "Skip API service"
}


component "Info storage service" as IDB{
component "Info db (elasticsearch)"  as IDB_
component "Load Balancer" as LB2
component "Directory" as DIR2

LB2->IDB_
LB2->DIR2
}


component "User storage service" as UDB{
component "User db (mySQL cluster)" as UDB_
component "Load Balancer" as LB3
component "Directory" as DIR3

LB3 -> UDB_
LB3 -> DIR3

}



component "External API" as EA{
component "weatherski" as AP1

component "skiapi" as AP2

component "onthesnow" as AP3

}


MSRV --> SA
SRV --> IDB
UREP -> UDB


COL --> EA
COL --> IDB
AGG --> IDB



WS --> SRVS
DAPI --> SRVS
WS --> DAPI


skinparam monochrome true
skinparam shadowing false
skinparam defaultFontName Courier
@enduml
```


Deployment view:

```puml
@startuml
!include <C4/C4_Container>
!include <C4/C4_Context>
!include <C4/C4_Component>



System_Boundary(boundary1, "user device"){
Container(mobileApp, "mobile app", "") {
  }

}

System_Boundary(boundary2, "User DB server"){

Container(userDb, "User Database container", ""){
}
}


System_Boundary(boundary3, "Info DB server"){

Container(info, "Info Provider", ""){

}
}






System_Boundary(boundary4, "API server"){
Container(api, "Skip API", "") {
  }

}




System_Boundary(boundary5, "Aggregator API server"){

Container(aggregator, "aggregator API", "") {

}
}






System_Boundary(boundary6, "Collector server"){
    Container(collectors, "collector API", ""){

}

}




System_Boundary(boundary8, "External server 1"){
Container(EAPI1, "external API 1", "") {
  }

}


System_Boundary(boundary9, "External server N"){

Container(EAPIN, "external API N", "")
}


System_Boundary(boundary10, "Dev API server"){

Container(DAPI, "Dev API", "")
}


System_Boundary(boundary11, "dev device"){
Container(browser, "dev api FE", "") {
  }
}



System_Boundary(boundary12, "watchdog server 1"){
Container(watchdog1, "watchdog 1", "") {
  }
}

System_Boundary(boundary13, "watchdog server 2"){
Container(watchdog2, "watchdog 2", "") {
  }
}

interface "services and APIs of the system (less arrows in uml)" as SRVS


Rel(collectors, info, "web")
Rel(collectors, EAPI1, "web")
Rel(collectors, EAPIN, "web")

Rel(aggregator, info, "web")


Rel(api, info, "web")

Rel(watchdog1, watchdog2, "RPC")
Rel(watchdog2, watchdog1, "RPC")
Rel(watchdog2, DAPI, "RPC")
Rel(watchdog1, DAPI, "RPC")
Rel(watchdog1, SRVS, "")
Rel(watchdog2, SRVS, "")
Rel(DAPI, SRVS, "")


Rel(api, userDb, "RPC")
Rel(browser, DAPI, "web")


Rel(mobileApp, api, "web")
@enduml
```


Process view 1, visualizing service availability:

```puml
@startuml
participant "Dev" as D
participant "Dev API" as A
participant "Storage" as SR
participant "Monitor" as M
participant "Service 1" as S1

M -> M : timeout
M -> S1 : send request
S1 -> M : response
M -> SR: store response time
D -> A : get uptime of service 1
A -> S : query data of component 1
S -> A : data of component 1
A -> D : uptime of component 1



@enduml
```

Process view 2, writing and starting DSL collector:

```puml
@startuml
participant "Dev" as D
participant "Dev API" as A
participant "Collector API" as CA
participant "Manager" as M
participant "Transpiler" as T
participant "Collector directory" as DIR


D -> A: add new collector as DSL
A -> CA: add new collector as DSL

CA -> M: add new collector as DSL

M -> T: transpile collector
T -> M: transpiled collector


M -> DIR : write DSL collector
M -> DIR : write transpiled collector

M -> CA : added collector

CA -> A: ok
A -> D: ok






D -> A:  start collector


A -> CA: start collector
CA -> M: start collector

M -> DIR: read collector
M -> M: start collector process
M -> CA: started collector


CA -> A: ok
A -> D: ok




@enduml
```


## 3
For what concerns our data sources:

We have designed our system to witstand breaking API changes: each API has
a different dedicated collector for each type of data we extract from it so that
would be the only thing to rewrite. Writing a new collector should also be faster
 due to the dedicated DSL.

We have put in place a system to monitor issues of this type with the collector
manager storing informations about the operation of each collector which are then
made available to the developer through the dedicated API.


The application wouldn't suddently break but, at worse, potentially serve
obsolete data if nothing better is available in the Information database.

We don't depend on other APIs at runtime and changes in those we use to deploy
our services would be discovered at deployment time.
