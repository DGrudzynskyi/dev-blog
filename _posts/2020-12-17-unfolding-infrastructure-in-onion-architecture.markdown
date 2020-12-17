---
layout: post
title:  "Unfolding infrastructure in onion architecture"
date:   2020-12-17 19:15:49 +0200
categories: architecture
tags: architecture, onion, ports-and-adapters, hexagonal, layered, coreutils, clean, infrastructure
---

### 1. Where to place my code?
**Through the mine developer experience, every rails-style application (.net MVC, spring MVC, whatever) had a folder (or project), called “Utils”. Or “Tools”. Or “Helpers”.**
Or some other very generic name with unknown purpose – up until I had opened this project and looked onto the code. Sometimes there are two or more projects like that, written by different generations/teams of software developers. Application might be formally split into layers following ‘layered’ or ‘onion’ architecture, but these folders are referenced from everywhere. And sometimes they want to reference each other, which is no-op due to potential circular dependency.

**Through the mine developer experience, I have argued a lot about the necessity of application architecture. Occasionally, developers just deny the need of architecture by means, wider than the chosen framework apply.**
This happens for a good reason: frameworks usually have detailed manuals explaining how to fit the simple application in it. In the opposite side of spectrum, description of _layered_ or _onion_ or _ports-and-adapters(hexagonal)_ architectures just gives us a wide picture and require making some project-specific decisions. Worse than that, some of these decisions are counter-intuitive and have obvious flaws, but very abstract gain, which might or might not be observed in far future.
What is common about two paragraphs above? 

One of such decisions is the decision about the code, highly reused by ‘whole’ application including the ‘domain’ objects.

- This is what drives the creation of ‘Utils’/’Tools’/’Helpers’ projects by people who try to be pragmatic.
- This is what drives an interfaces nightmare and makes newcomers mad about the thousands of classes if application architect is a purist, trying to keep domain model independent of anything.

In this article I am talking about approaching this task in _layered_, _onion_ and _ports and adapters_ architectures. I will start from the _layered_ architecture (which is considered being outdated nowadays) and make a transition into more modern _onion_ and _ports and adapters_. 
The purpose of the article is to eliminate an uncertainty in answering to ‘where should I place my commonly reused code?’.

### 2. Layered architecture. ‘Infrastructure’:
Let’s look on a couple of definitions:

<div class="block-with-image-container block-with-image-container--narrow">
  <img 
    src="/assets/2020-12-17/layered.png" 
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
    src="/assets/2020-12-17/palermo-layers.png" 
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
      src="/assets/2020-12-17/layered-seeman.png" 
      alt="Mark Seeman in the “Dependency Injection in .NET”, chapter 2, draw layers without something called infrastructure" 
      class='left-column-image left-column-image--medium'
  />
    <p>
      Mark Seeman in the “Dependency Injection in .NET”, chapter 2, draw layers without something called “infrastructure”, effectively bypassing this piece of the software as well. He only focusing on analysis of ‘data access’ as a crucial piece of infrastructure.
    </p>
    <p>
The quick essence of that chapter is given in the Mark’s <a herf='https://blog.ploeh.dk/2013/12/03/layers-onions-ports-adapters-its-all-the-same/'>article</a>. This article also nicely aligns <i>layered</i>, <i>onion</i>, and <i>ports and adapters</i> architectures, so i recomment you to read it before proceeding with current article.
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
    src="/assets/2020-12-17/ports-adapters.png" 
    alt="Ports and Adapters (Hexagonal) architecture by Alistair Cockburn" 
    class='left-column-image left-column-image--medium'
    />
  <p>
    Read an original <a hreh='https://alistair.cockburn.us/hexagonal-architecture/'>article published by Alistair Cockburn:</a> 
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
      src="/assets/2020-12-17/palermo-onion.png" 
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
      src="/assets/2020-12-17/CleanArchitecture.jpg" 
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

### 4. Have we caught all our boundaries in the outer-most circle of onion? 
By definition of onion, our core (domain model) has no dependencies.

But does it?

Let me once again cite Robert Martin from another <a href='https://blog.cleancoder.com/uncle-bob/2015/08/06/LetTheMagicDie.html'>post</a>:

> The authors of rxJava, and of Spring, and JSF, and JPA, and Struts, and [put your favorite framework here] are all searching for the same thing. These frameworks are born out of frustration with the language; and are an attempt to improve upon that language.
Every framework you’ve ever seen is really just an echo of this statement:
-	My language sucks!

I would argue with the point that every framework is an attempt to improve the language – rather it is an attempt to adopt the language to programming models/principles, not present in this language natively. Such as spring has grown as DI framework/IoC container (and then suddenly grown into swiss knife of java world but who cares). RxJava is designed to account for the reactive programming. 

I am skeptical about including any of these things into the language itself, but you might accept the R. Martin’s definition. It does not matter. 

<div class="block-with-image-container block-with-image-container--medium">
  <img 
      src="/assets/2020-12-17/onion-with-language.png" 
      alt="Onion architecture mentioning language within the core" 
      class='left-column-image left-column-image--medium'
  />
  <p>
    What matters is that this quote might make you suspicious <i>whether your core is truly isolated as it is written using the language</i>. And this language has its features. Sometimes special or unique (like expressions in C# or pointers in C), sometimes – widely adopted by many different languages: (data structures support, dates management, class/interface inheritance). 
  </p>
  <p>
    And the core of application already depends on this set of features, so-called <b>language</b>. So, to be honest to himself, one should take the favorite sort of onion and put the <i>language</i> to the center of it. I am doing it to mine like that:
  </p>
  <p>
    There is no emphasis on the outer circles of onion. You may replace <i>Application Services</i> with <i>Use Cases/Ports</i> if it seems more relevant for you. You may split <i>Domain model</i> into <i>Domain Entities/Domain Services</i>.
  </p>
</div>

### 5. What if my language Sucks?

We are freely using the API provided to us by our _language_. And all layers of application toughly bound to this API.
Then what if some _essential_ or _dumb as heck_ features are missing from language?  
<quote>“Let’s talk about JavaScript” (©. Gary Bernhardt)</quote>

Do you really want to abuse the constructors of your domain models with the _IArrayUtils_ or _IDateUtils_ function libraries only because JavaScript is unable to natively add dates or join arrays?
There are two downsides with extracting such function sets into interfaces:
1.	Adding extra complexity. The more logical items you have the more complexity you have. 
2.	Such _stateless/dependency-less_ services are mixed up with stateful dependencies and other dependencies, which assuming application boundary invocation. As soon as there are more than 5-6 dependencies in the service – its constructor become hard to read. So, it is hard to quickly understand the responsibility of service by analyzing its dependencies.
3.	It’s become even harder because our _logical dependencies_ being messed up with the _language patches_, only needed to add an essential feature to the language itself.

Then the common benefits of DIP are negated:
1.	Ability to replace an implementation of interface – negated by the fact that as soon as you developed patch to language (such as add function for adding dates) and test it - there are no more meaningful reasons for changing it other than performance. And if (for some reason) you find out that performance should be boosted – there are no reason to re-write all method in the IDateUtils and replace it. Rather, single method to be optimized within the same class.
2.	Ability to re-configure the dependencies of these utility classes without falling into poor man DI – negated by the fact that there are zero dependencies in such _language patch_ function libraries.

From there, I came up to the conclusion that for the _language patches_ I don’t want to extract the implementations to the outer circle of the onion because it harms more than helps. I don’t want to define interfaces either as these items have no particular reason for change.

I would call the set of such patches to language as a **CoreUtils**. Might not be the best name ever, so feel free to suggest the better naming in comments. 

**CoreUtils** is what I would place instead of the _‘and so on’_ part in the Eric Evans’s description of layered architecture. So if we compare _infrastructure_ of the layered architecture with the _pieces of onion_ we would end with:

|_Infrastructure_ parts in layered architecture|Corresponding _onion_ parts|
|---|---|
|Code, related to application boundaries|Interface in the _inner circles_ of onion, implementation in the _outer circle_ of onion|
|Pattern of interaction between the four layers of framework|IoC container usage and placing composition roots in the outer circles of onion.|
|Commonly reused functions, neither related to boundaries, nor to interaction between layers|CoreUtils|











Where `YEAR` is a four-digit number, `MONTH` and `DAY` are both two-digit numbers, and `MARKUP` is the file extension representing the format used in the file. After that, include the necessary front matter. Take a look at the source for this post to get an idea about how it works.

Jekyll also offers powerful support for code snippets:

{% highlight ruby %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
