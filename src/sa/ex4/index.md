## Sustainability vs. Ease of support

Depending on the complexity of use and importance of a service for a user
providing strong and reliable support can be a necessity for its success.
There are different ways of providing support with vastly different costs. For
instance Apple has set up a support network through its many shops while Adobe
has support available only on a call service.

Maintaining support for the duration of the service clearly incours in running
costs that can ultimately hurting the sustainability of it so the degree of needed
support must clearly be estimated before deplyment.

In our case the application should be very easy to use and at the same time a provide
a service that, while useful, is not absoulutely necessary for the customer. We thus
opt to provide a very cheap for of support limited to mails. While this might lose us
some customers we evaluate the spared resources to be worth the tradeoff.



## Design consistency vs. Time to market (feasibility)

For some applications a short time to market is a sure key to success even if
the end result is a software full of bugs with a messy codebase. For instance
game based off an instalment of a famous saga are developped on a very short time
window and as a result are in most cases at best mediocre and riddled with bugs,
still riding the trend the manage to sell a fait amount of copies given the employed
resources.

In our case while releasing the application before the snow season starts would
be preferable sacrificing good practices for a quick and timely release would not
be a good tradeoff especially when we consider that our service will need to be
maintened and fixing ater its deployment would be far more expensive.

In some cases (especially for large pallications) design consinstency and feasibility
might actually be synergistic in nature since adhering to precisely established
conventions and good practices will make code easier to work on.


## Interoperability vs. simplicity

We will need to gather as much data as possible from various sources about weather
and snow conditions.
Since there is no single API that provides this service we will need to aggregate
data from multiple sources including what we receive from our users.

This will inevitably increase the complexity of our project as we will need
sift our data through many levels to clean and filter it but it's a worthy tradeoff
when we consider how much it benefits the utility of our service.
