
## Container view

```puml
@startuml
!include <C4/C4_Container>
!include <C4/C4_Context>
!include <C4/C4_Component>

Person(user, "app user", "")

System_Boundary(boundary, "skip"){



Container(userDb, "User Database container", "MySQL", ""){
Component(db1, "User Database container", "MySQL", "Contains data about the users")
}


Container(info, "Info Provider", "Docker", ""){
Component(db2, "Info Database", "Elasticsearch", "Contains data about the resorts")
Component(rp1, "Reverse proxy", "Kotlin", "if possible uses cache for expensive operations")
Component(cache1, "cache db", "Kotlin", "caches reusults of expensive operations")

}




Container(api, "Skip API", "Docker") {
    Component(controller, "controller", "MVC: controller", "Kotlin")
    Component(service, "service", "MVC: model", "Kotlin" )
  }



Container(aggregator, "aggregator container", "Docker") {
    Component(aggregatorComp, "aggregator component", "Python", "aggregates content for the service")
  }



Container(collector, "collector container", "Docker") {
    Component(collectorComp1, "collector 1", "Python", "collects data from sources")
    Component(collectorComp2, "collector 2", "Python", "collects data from sources")
    Component(collectorCompN, "collector N", "Python", "collects data from sources")
  }


Container(mobileApp, "mobile app", "External Device") {
    Component(UI, "UI", "JS", "")
  }

Container(EAPI1, "external API 1", "external device", "info soruce 1") {
    Component(UI, "UI", "JS", "")
  }

Container(EAPIN, "external API N", "external device", "info source N") {
    Component(UI, "UI", "JS", "")
  }


Rel(user, mobileApp, "uses to watch resort info")

Rel(collector, info, "ingests to")
Rel(collector, EAPI1, "fetches data from")
Rel(collector, EAPIN, "fetches data from")

Rel(aggregator, info, "fetches data from and posts aggregate to")


Rel(api, info, "fetches aggregated info from")
Rel(api, userDb, "fetches from and posts user data to")

Rel(mobileApp, api, "exchanges data with, certifies credentials from")





}
@enduml
```







## Deployment view



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




System_Boundary(boundary5, "Aggregator server"){

Container(aggregator, "aggregator", "") {

}
}






System_Boundary(boundary6, "Collector server"){
    Component(collectors, "collectors", ""){
    Component(collectorComp1, "collector 1", "")
    Component(collectorComp2, "collector 2", "")
    Component(collectorCompN, "collector N", "")
}

}




System_Boundary(boundary8, "External server 1"){
Container(EAPI1, "external API 1", "") {
  }

}


System_Boundary(boundary9, "External server N"){

Container(EAPIN, "external API N", "")
}





Rel(collectors, info, "web")
Rel(collectors, EAPI1, "web")
Rel(collectors, EAPIN, "web")

Rel(aggregator, info, "web")


Rel(api, info, "web")
Rel(api, userDb, "RPC")

Rel(mobileApp, api, "web")
@enduml
```






## Deployment strategy ADR

1. What did you decide?

What protocol skip should use to release a new version.

2. What was the context for your decision?

New releases are a critical moment in the life of an application, a bad release
can easily cause users to migrate to competitors. At the same time users expect
applications to improve over time and might not even agree that new features
are improvements.


3. What is the problem you are trying to solve?

We don't want to compromise the availability and reliability of our application
when releasing a new version. At the same time we want to avoid spending a
major amount of resources if possible.

4.  Which alternative options did you consider?

* canary
* shadow
* gradual pahse-in
* A/B
* Blue/Green
* Big Bang
* pilot


5. Which one did you choose?

canary -> gradual phase-in: front-end
back-end: canary

6. What is the main reason for that?

We want to give users some flexibility to update their application, at the same
time we don't want to confuse them with ulterior options to rollback their changes
and, at some point, we want all of them to move to the new version.

A gradual phase-in (obviously preceeded by testing with pilots) adapts quite well
to this requirements and doesn't need us to develop extra infrastructure. A
deployment with a canary approach might still be risky as canary users might
carry some bias which is why we complement it with a gradual phase-in.


The A/B method could be emplyed if there are strong doubts between two differently
designed versions of our applications but it might prove to be expensive to enact
and should be used carefully.


For the backend we are mostly interested in the fact that new updates won't
contain bugs and break the application so even users with biases will still work
which is why we go with a canary approach. With more resources we could compare
results users obtain from the new API with the old one via a shadow approach
but it would likley require tons of extra work.
