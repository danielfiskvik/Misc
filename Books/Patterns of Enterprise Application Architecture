# Patterns of Enterprise Application Architecture
##### By Martin Fowler (2002)

## Chapter 1: Layering

A basic example of layering: FTP < TCP < IP < Ethernet

Benefis of layering:
* You can understand a layer without knowing much about the others.
* Minimize dependencies.
* When a layer’s built, you can use it for higher-level services.
* Layer equivalent substitution.

The 3 Principal Layers:
1. Presentation
2. Data Source
3. Domain

Keep them separate in some way -- it reduces coupling.

The data source and domain layers should not depend on the presentation layer.

## Chapter 2: Organizing Domain Logic

3 Primary Patterns:

1. Transaction Script
 - Separate domain logic into transactions. (Add item to cart, check out, etc.)
2. Domain Model
 - Separate into models corresponding to nouns involved.
3. Table Module
 - Like Domain Model but only has one object, rather than one for each logical object. Deals with Record Sets.

Service Layer:
   * An extra layer over top of the Domain Model / Table Module.
   * The presentation interacts with the domain logic through the service layer.
   * Good place to put things like transaction control and security.
   * Can basically be used as a Facade for the lower-level objects, only responsible for forwarding calls and data to them. (Often convenient because it ends up being oriented around use cases.)
   * Fowler: “My preference is thus to have the thinnest Service Layer (133) you can, if you need one.”

## Chapter 3: Mapping to Relational Databases

### Architectural Patterns
“It’s wise to separate SQL access from the domain logic and place it in separate classes.”

### The Gateway Pattern
   * Gateway: Holds and stores data pulled from the database.
   * Row Data Gateway: One gateway per object/record. (Basically Kohana ORM.)
   * Table Data Gateway: One gateway per table; holds data for all records found.

### Another option: Data Mapper

*“A better route is to isolate the Domain Model (116) from the database completely, by making your indirection layer entirely responsible for the mapping between domain objects and database tables.”*

Data Mapper: Handles all of the loading and storing between the database and the Domain Model.

### The Behavioral Problem

How do you deal with many (database entity) objects in granular faction without performing many SQL queries?

Solution:
   * The unit of work pattern. Unit of work registers changes, so that they can be batch updated (as a single unit of work) at the appropriate time.


Problem of concurrency -- How do you handle when your application needs to access the same record in multiple places, and be sure that updates in one place are reflected in the others?
   * An identity map, Which registers the entities which have been loaded so that they can be accessed elsewhere in your app, always showing the latest, most updated version of the entity.
   * Also doubles as a database cache, reducing the number of SQL calls required. (However, database performance is not the primary concern; maintaining correct identities is.)

If you employ a method of pulling in linked objects from the database, such as ORM, how do you optimize so that you are not pulling in huge amounts of data when they're not needed?
   * The lazy loader pattern. Instead of loading the linked objects, load Lazy Loaders which can load them on demand.

### Reading In Data
   * Try to pull back multiple rows at once. Be liberal. Pulling back potentially too many rows, one time, is more efficient than only pulling in what’s necessary, multiple times.

### Structural Mapping Patterns

#### Mapping Relationships
Relationships between object in the database are inherently represented backwards from the way they are represented in memory. Database: foreign key column holds a key, and only one of them. Memory: object holds the actual linked object, not just a key; object can hold an array of them, not just one. 

Solutions:
   * Identity Field
   * Identity Map

Lots of relationships to small objects can cause a bottleneck with many DB calls. Try using a Serialized Large Object (for example a field with a JSON object) to reduce DB queries.

## Chapter 4: Web Presentation

### Model View Controller Pattern
Encapsulates the idea of separation of concerns. In order to change the user interface, the view and "input controller" can stay the same; just edit the view.

> "The first, and most important, reason for applying Model View Controller (330) is to ensure that the models
are completely separated from the Web presentation."

#### Input controller vs. Application controller:
 - Input controller: "pulls information off the request. It then forwards the business logic to an appropriate model object."
 - Application controller: "The purpose of an Application Controller (379) is to handle the flow of an application, deciding
which screens should appear in which order."

> "Not all systems need an Application Controller (379). They're useful if your system has a lot of logic about
the order of screens and the navigation between them."

#### View Patterns
1. Template View: Use a template with markers where dynamic content needs to go. (Can be messy if you're just directly inserting PHP or whatever programming languages into the template. Better to use a templating language like Mustache.)
2. Transform View: Transforms one representation (e.g. XML) into another (HTML?)
3. Two Step View: Not necessarily an alternative to the other two, but a mixin. Template and Transform views are normally 1-step views. A two-step view will:
  1. Take the model and transform it into a "logical screen".
  2. Take the logical screen and transform it into the actual view (e.g. HTML).

Advantages of using a two-step view:
1. Centralize what kind of markup is used into one place.
2. Makes white-labeling easy.
3. Can easily serve different pages to different devices. (e.g. Mobile vs non-mobile)

#### Input Controller Patterns
1. Page Controller: Represents a single (web) page. Finds the appropriate models, instantiates the right view(s) and displays the resulting page. Contrasts with...
2. Front Controller: One object that handles *all* requests and creates a separate object to process it. "Allows you to centralize all HTTP handling within a single object".

## Chapter 6: Session State

### The Value of Statelessness

“Stateless server”: a server that does not retain state between requests.

**Example:** The user provides an ISBN number and the server returns a book’s info.
 - Stateless: Server returns book info.
 - Stateful: Server returns info and keeps track of all ISBN numbers requested by this IP.

The primary issue with stateful servers: Server Resources.
 - They have to store the state somewhere!
 - For high-traffic sites, this means a lot of storage.

“So everything should be stateless, right?” That would help, but some interactions with the user are inherently stateful. (e.g. Shopping carts)

> The good news is that we can use a stateless server to implement a stateful
session; the interesting news is that we may not want to.

### Session State

State affects *consistency*. The data in a state may not be valid.
 -Customer edits a setting; input is not valid, server responds and tells them so, but the value is still part of the state at this point.

State affects *isolation*. A state may pull persistent information (e.g. user's information) and the information may change while the session is still active, thus rendering it out of date.

#### Ways to store session state

1. **Client Session State** (e.g. Holding state in URL, cookies, hidden form field, local storage.)
 - Con: The session must be sent with every request. (So only use if state will be small.)
 - Be concerned about security; unless encrypted, the user can get his hands on this data and edit it.
 - Pro: If there are lots of users that simply disappear in the middle of a session without letting you know, server resources have not been sacrificed. (But this is easily fixed with a session timeout.)
2. **Server Session State** (Holding state in memory on the server; local filesystem, in database as serialized object.)
 - Better performance than Database Session State if there are lots of active users.
 - Pro: Usually requires less development time than the other two because the session object does not need rebuilt from pieces; it's serialized.
3. **Database Session State** (State exists in database, but not just as serialized object; there is a schema for holding the data.)
 - Good for when most users are idle. (Doesn't require much in the way of DB resources.)

**Session migration:** Dispersing a session to all redundant servers in case the user accesses a different one next request.

**Server affinity:** Forcing the user to be sent to the same server each request, in order to keep the session accessible.
 - Careful, it's probably necessary to identify the user by IP, and the server won't be able to distinguish between different users on the same IP.

All 3 approaches can be mixed to store different parts of session data. (More complicated though.)
