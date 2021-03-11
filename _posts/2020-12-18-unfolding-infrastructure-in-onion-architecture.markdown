---
layout: post
comments: true
title:  "Unfolding infrastructure in the Onion architecture"
date:   2020-12-18
categories: architecture
tags: architecture, onion, ports-and-adapters, hexagonal, layered, coreutils, clean, infrastructure
---

- _Stuck trying to figure out where to place your code without tricking an existing **software architecture**?_
- _Feels like presence of the **architecture** drives an embarrassment across the team?_
- _Project's codebase just screams about complete rewrite due to an outdated dependencies? What if presence of the **architecture** could have saved it..._

If you've ever found yourself frustrated with any of the above items - this article is for you.

### 1. Preface: where to place my code?
Throughout my developer career, every rails-style application (.net MVC, spring MVC, whatever) has a folder/project called “Utils”. Or “Tools”. Or “Helpers”.
Or some other very generic name with unknown purpose – up until I open this project and look onto the code. Sometimes there are two or more projects like that, written by different generations/teams of software developers. Application might be formally split into layers following _layered_ or _onion_ architecture, but these folders are referenced from everywhere. And sometimes they want to reference each other, which is no-op due to potential circular dependency.

Throughout my developer career, I have argued a lot about the necessity of application architecture. Occasionally, developers just deny the need of architecture by means, wider than the chosen framework apply.
This happens for a good reason: frameworks usually have detailed manuals explaining how to fit a simple application in. In the opposite side of spectrum, description of _layered_ or _onion_ or _ports-and-adapters(hexagonal)_ architectures just gives us a wide picture and require making some project-specific choices. Worse than that, some of these choices are counter-intuitive and have obvious flaws but very abstract gain, which might or might not be observed in far future.

What is common about two paragraphs above? 

One of such choices is the decision about the code, highly reused by _whole_ application including the _domain_ objects.

- This is what drives the creation of ‘Utils’/’Tools’/’Helpers’ projects by the people who try to be pragmatic.
- This is what drives an interfaces nightmare and makes newcomers mad about the thousands of classes in case application architect is a purist, who is trying to keep the domain model isolated from anything.

In this article I am approaching this task in _layered_, _onion_ and _ports and adapters_ architectures. I will start from the _layered_ architecture (which is considered being outdated nowadays) and make a transition into more modern _onion_ and _ports and adapters_. 
The purpose of the article is to eliminate an uncertainty while answering to _“where should I place my commonly reused code?”_.

### 2. Layered architecture. ‘Infrastructure’:
Let’s look on a couple of definitions:

<div class="block-with-image-container block-with-image-container--narrow">
  <img 
    src="{{site.baseurl}}/assets/2020-12-17/layered.png" 
    alt="From Eric Evans book “Taking Complexity into the heart of the software” (page 74):" 
    class='left-column-image left-column-image--small'
    />
  <p>
  The description of infrastructure layer from Eric Evans book “Taking Complexity into the heart of the software” (page 74):
  </p>
  <blockquote>“Provides generic technical capabilities that support the higher layers: message sending for the application, persistence for the domain, drawing widgets for the UI, and so on. The infrastructure layer may also support the pattern of interactions between the four layers through an architectural framework.” © 
  </blockquote>
</div>

<div class="block-with-image-container block-with-image-container--narrow">
  <img 
    src="{{site.baseurl}}/assets/2020-12-17/palermo-layers.png" 
    alt="Jeffrey Palermo, describing layers in his initial article introducing term ‘onion’. https://jeffreypalermo.com/2008/07/the-onion-architecture-part-1/" 
    class='left-column-image  left-column-image--small'/>
  <p>
  Jeffrey Palermo describes layered architecture in his initial <a href='https://jeffreypalermo.com/2008/07/the-onion-architecture-part-1/ '>article introducing term ‘onion’</a>. 
  </p>
  <p>
  Here, domain persistence is explicitly extracted to <i>data</i> layer. However, Jeffrey is not giving any descriptive meaning to infrastructure stating: 
  <blockquote>
  “I am intentionally ignoring infrastructure here because this typically varies from system to system.” ©
  </blockquote>
  </p>
</div>

<div class="block-with-image-container block-with-image-container--medium">
  <img 
      src="{{site.baseurl}}/assets/2020-12-17/layered-seeman.png" 
      alt="Mark Seeman in the “Dependency Injection in .NET”, chapter 2, draw layers without something called infrastructure" 
      class='left-column-image left-column-image--medium'
  />
    <p>
      Mark Seeman in the “Dependency Injection in .NET”, chapter 2, draw layers without something called “infrastructure”, effectively bypassing this piece of the software as well. He only focusing on analysis of <i>data access</i> as a crucial piece of infrastructure.
    </p>
    <p>
The quick essence of that chapter is given in the Mark’s <a href='https://blog.ploeh.dk/2013/12/03/layers-onions-ports-adapters-its-all-the-same/'>article</a>. This article also nicely aligns <i>layered</i>, <i>onion</i>, and <i>ports and adapters</i> architectures, so i recommend you to read it before proceeding with current article.
    </p>
</div>

Let's deconstruct an original Eric Evans definition of _Infrastructure_:
-	Message sending for the application
-	Persistence of the domain
-	Drawing widgets for the UI
-	Pattern of interaction between the four layers of framework
-	‘and so on’ – this is the most important statement here. Every function reusable across all layers ends up here. 
Note that other authors may skip talking about infrastructure layer at all, even though the data access is extracted as a separate layer. This forces some teams end up with an _infrastructure_ turned into a _trash can_. 

To sum up, there are:

- **The good news:** you know where to put your generic functions.
- **The bad news:** you put them into the _trash can_. Now you have a part of application, which depends on all the external API’s and low-level communication details. Subsequently your application _from-top-to-bottom_ depends on this trash can. This makes impossible to painlessly replace any of the application integrations.

### 3. ‘Ports and Adapters’ architecture. What is there?

<div class="block-with-image-container block-with-image-container--medium">
  <img 
    src="{{site.baseurl}}/assets/2020-12-17/ports-adapters.png" 
    alt="Ports and Adapters (Hexagonal) architecture by Alistair Cockburn" 
    class='left-column-image left-column-image--medium'
    />
  <p>
    Read an original <a href='https://alistair.cockburn.us/hexagonal-architecture/'>article published by Alistair Cockburn:</a> 
    It emphasizes the importance of keeping an application separate from the <b>external boundaries</b> and provides a comprehensive description of what to be considered as such boundary.
  </p>
  <p>
Ports and adapters do not care about the inner structure of your application. So, this article defines only the fact that every single external boundary is <i>referencing and application</i> instead of <i>application referencing external boundaries</i>. This way we achieve application robustness as any of the boundaries might be replaced by re-implementing ports or adapters.
  </p>
</div>


Note how the majority of items, listed as _infrastructure_ responsibilities in _layered_ architecture are now listed as external boundaries:
-	Message sending for the application – external boundary (notifications)
-	Persistence of the domain – external boundary (database)
-	Drawing widgets for the UI – external boundary (GUI)
-	~~Pattern of interaction between the four layers of framework~~ – eliminated as this architecture is not concerned about inner application structure.
-	‘and so on’ – **Boom**. What if we have a generic function, which is not related to application boundary? I am going to elaborate it later.

So far you may realize that although the _ports and adapters_ does not give us an explicit answer about _where to place our generic logic_, it provides a good guidance about narrowing such generic logic to everything _except_ application boundaries, while every boundary requires its very own, unique and separated adapter.


### 4. ‘Onion’ architecture. Getting rid of transitive dependencies:


<div class="block-with-image-container block-with-image-container--medium">
  <img 
      src="{{site.baseurl}}/assets/2020-12-17/palermo-onion.png" 
      alt="Onion, defined by Jeffrey Palermo" 
      class='left-column-image left-column-image--medium'
  />
  <p>Check the initial <a href='https://jeffreypalermo.com/2008/07/the-onion-architecture-part-1/'>article</a> published by Jeffrey Palermo with the description of an <b>onion</b> architecture:
  </p>
  <p> 
Its intention is to get rid of <i>transitive dependencies</i> (with explicit focus on dependency from UI to DB).
It is doing so by applying the dependency inversion by same principles, described in the <i>ports and adapters</i>. Interfaces are defined in the application core and implemented by the application <i>boundary adapter</i>.
  </p>
  <p>
Note, that <i>infrastructure</i> is still present as a fragment of external onion circle. I guess it is done to make things easier to read by someone, used to layered approach. Infrastructure is visually broken into pieces, all of those are application <i>boundaries</i>. Other parts of outer circle (UI/Tests) are also application boundaries.
  </p>
</div>

Some authors unfold the _infrastructure_ in onion architecture and provide the more comprehensive and less _layered-oriented_ kind of onion. 

<div class="block-with-image-container block-with-image-container--medium">
  <img 
      src="{{site.baseurl}}/assets/2020-12-17/CleanArchitecture.jpg" 
      alt="Clean Architecture by Robert Martin" 
      class='left-column-image left-column-image--medium'
  />
  <p>
  One of the most well-known examples of such unfolding is given in Robert Martin’s <a href='https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html'>Clean architecture</a> article. It fits the <i>ports and adapters</i> into the mental model of <i>onion</i> by the way unfolding the <i>infrastructure</i> into the set of potential <i>application boundaries</i>: Devices, DB, External interfaces e.t.c. 
  </p>
</div>


Note that onion architecture does care about the inner application structure. And it describes some of the infrastructure pieces from layered architecture as an application boundaries. But not *all* of them. We still have some pieces, originally mentioned as part of <i>infrastructure</i> but not being an <i>application boundary</i>:

-	Pattern of interaction between the four layers of framework – this is handled using dependency injection plus extraction of the **composition roots** to the outer most circle.
-	‘and so on’ – Well, here is why this article is written :)

### 4. Have we caught all boundaries in the outer-most circle of onion? 
By definition of the _Onion_, our **core** (domain model) has no dependencies.

But does it?

Let me once again cite Robert Martin from another <a href='https://blog.cleancoder.com/uncle-bob/2015/08/06/LetTheMagicDie.html'>post</a>:

> The authors of rxJava, and of Spring, and JSF, and JPA, and Struts, and [put your favorite framework here] are all searching for the same thing. These frameworks are born out of frustration with the language; and are an attempt to improve upon that language.
Every framework you’ve ever seen is really just an echo of this statement:
-	My language sucks!

I would argue with the point that every framework is an attempt to improve the language – rather it is an attempt to adopt the language to programming models/principles, not present in this language natively. Such as Spring has grown as DI framework/IoC container (and then suddenly grown into swiss knife of java world but who cares). RxJava is designed to account for the reactive programming. 

I am skeptical about including any of these into the language itself, but you might accept the R. Martin’s definition. It does not matter. 

<div class="block-with-image-container block-with-image-container--medium">
  <img 
      src="{{site.baseurl}}/assets/2020-12-17/onion-with-language.png" 
      alt="Onion architecture mentioning language within the core" 
      class='left-column-image left-column-image--medium'
  />
  <p>
    What matters is that this quote might make you suspicious <i>whether your core is truly isolated</i>. It is written using the <b>language</b>. And this language has its features. Sometimes special or unique (like expressions in C# or pointers in C), sometimes – widely adopted by many different languages: (data structures support, dates management, class/interface inheritance). 
  </p>
  <p>
    And the core of application already depends on this set of features, so-called <i>language</i>. So, to be honest to himself, one should take the favorite sort of the onion and put the <i>language</i> to the center of it. I am doing it to mine like that:
  </p>
  <p>
    There is no emphasis on the outer circles of onion. One may replace <i>Application Services</i> with <i>Use Cases/Ports</i> if it better suites the application. One may split <i>Domain model</i> into <i>Domain Entities/Domain Services</i> as well.
  </p>
</div>

### 5. What if my language sucks?

We are freely using the API provided to us by our _language_. And all layers of application are toughly bound to this API.
Then what if some _essential_ or _dumb as heck_ features are missing from language?  
<quote>“Let’s talk about JavaScript” (©. Gary Bernhardt)</quote>

Do you really want to abuse the constructors of your domain models with the _IArrayUtils_ or _IDateUtils_ function libraries only because JavaScript is unable to natively add dates or join arrays?
There are two downsides with extracting such function sets into interfaces:
1.	Adding extra complexity. The more logical items you have the more complexity you have. 
2.	Such _stateless/dependency-less_ services are mixed up with stateful dependencies and other dependencies, which assumes the application boundary invocation. As soon as there are more than 5-6 dependencies in the service – its constructor become hard to read. So, it is hard to quickly understand the responsibility of the service by analyzing its dependencies.
3.	It’s become even harder because our _logical dependencies_ are being messed up with the _language patches_, only needed to add an essential feature to the language itself.

Then the common benefits of DIP are negated:
1.	Ability to replace an implementation of interface – negated by the fact that as soon as you developed patch to language (such as add method for adding dates) and test it - there are no more meaningful reasons for changing it other than performance. And if (for some reason) you find out that performance should be boosted – there is no reason to re-write whole implementation of the IDateUtils. Rather, the method to be optimized within the already created class.
2.	Ability to re-configure the dependencies of these utility classes without falling into poor man DI – negated by the fact that there are zero dependencies in such _language patch_ function libraries.

From there, I came up to the conclusion that for the _language patches_ I don’t want to extract implementations to the outer circle of the onion because it harms more than helps. I don’t want to define interfaces either as these items have no particular reason for change.

I would call the set of such patches to language as a **CoreUtils**. Might not be the best name ever, so feel free to suggest the better naming in comments. 

**CoreUtils** is what I would place instead of the _‘and so on’_ part in the Eric Evans’s description of layered architecture. So if we compare the _infrastructure_ of the layered architecture with the _pieces of onion_ we would end with:

|_Infrastructure_ parts in layered architecture|Corresponding _onion_ parts|
|---|---|
|Code, related to the application boundaries|Interface in the _inner circles_ of onion, implementation in the _outer circle_ of onion|
|Pattern of interaction between the four layers of framework|IoC container usage with composition roots placed in the outer-most circle of onion.|
|Commonly reused functions not related to application boundaries or interaction between layers|CoreUtils|

### 6. CoreUtils – what to include?

<div class="block-with-image-container block-with-image-container--medium">
  <img 
      src="{{site.baseurl}}/assets/2020-12-17/onion-with-core-utils.png" 
      alt="Onion architecture mentioning language and utils within the core" 
      class='left-column-image left-column-image--medium'
  />
  <p>
    It is obvious that everything placed in the <i>CoreUtils</i> become carved in stone for an application. Whatever is placed here shall be changed as rare as the language version is being changed.
  </p>
  <p>
    Below is the list of criteria I use to move the functional to the <i>CoreUtils</i> project/folder. 
  </p>
  <p>
    It might be replaced by single statement <i>“you should only use pure functions respecting dependency rule”</i> but explicit list seems more descriptive to me.
  </p>
</div>


1.	Functional shall neither be special to the application nor to the platform this application runs within. So, nothing specific to _web_ or _desktop_ or _mobile_ or _sql_ or _blockchain_ or _finance_ or _medicine_.
2.	Functional must not be configurable. E.g., functions shall not behave differently in different environments.
3.	Functional must not know about any of the application boundaries. It subsequently should neither be an SDK for communications with external resources nor it shall use such SDK.
4.	The one I am not sure yet: functional should be stateless. Every need for specific data structure I have ever experienced was about adding this data structure as a part of my domain model or adding application-specific objects. However, I can think of the case when one might need to add some math extensions to JS or C# code. And such extensions might need special data structures (e.g. matrices). If you decide to add some data structure/stateful object to this layer – think twice whether it is platform-agnostic, domain-agnostic and is not going to be used as a configuration.
5.	You may reference external libraries as a part of ‘CoreUtils’ as long as they satisfy provided criteria. For example, for JavaScript: 
  -	lodash – yes. 
  -	DateFNS – yes. 
  -	MomentJS – no. Because it can be configured with the locale. 
  -	React – no. Because it is specific to UI rendering.


### 7. Epilogue: tiny example - working with dates in JavaScript.

Consider work with dates in regular JS SPA front-end. You may want to do adding/subtracting on dates, formatting the dates to human-readable form, formatting the dates to API-readable form, parsing the dates.

Adding/subtracting of dates is perfectly manageable within the _CoreUtils_. My choice is just to import the dateFNS library and use addDays function all over the application without any specific wrapper.
However, formatting the dates for API is an interaction with _API boundary_. And formatting the dates to user is an interaction with _UI boundary_. 
So, we would rather end up with two interfaces in the Application Services layer (note: much depends on your app. I can see _IUIDatesFormatter_ even being a part of Domain Services as domain logic might rely on _what user sees_ for some special apps):


{% highlight typescript %}
interface IAPIDatesFormatter {
    parseDate(apiStringifiedDate: string): Date;
    formatDate(date: Date): string;
}

interface IUIDatesFormatter {
    formatDate(date: Date, customFormat?: string);
    parseDate (uiDateRepresentation: string, customFormat?: string): Date;
}
{% endhighlight %}


_IAPIDateFormatter_ implementation must know how API wants to receive the date objects. Would it be UTC timestamp, or should it be sent with user’s timezone offset? With the offset, set in the global runtime configuration? It totally depends on a logic of the application and on a logic of the web server.

Formatting of the dates to user then remains totally unaffected by the decision made by the technical team working on the API. Instead, it may be driven by aesthetic feelings of the customer as well as by necessity to display dates in a timezone of user choice.

However, neither of these two services must become a part of the _CoreUtils_ because both highly depend on the _boundaries_, our application works within.

### Conclusion:

I hope this article helps you to develop proper coding discipline across the team and justify the necessity of additional interfaces for functions, related to application boundaries. 

I hope that presence of _CoreUtils_ in the solution helps you to avoid an excessive interfaces creation. This might be the way to reconcile between the ‘architecture purists’ and poor developers who just want to get the shit done.

Just make sure that guidance and restrictions for putting something into _CoreUtils_ are clear and accepted by whole team. It is extremely important in order to avoid falling into a burden, happened to the term _Infrastructure_ long time ago.