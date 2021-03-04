---
layout: post
title:  "Framework-agnostic browser-based SPA"
date:   2021-03-04
categories: architecture
tags: architecture, js, ts, javascript, typescript, ports-and-adapters, hexagonal, layered, onion, mobx, react
---

### 1. But... why? 

- There are a lot of frontend frameworks.
- There are a lot of tutorials and documentation showing how to build an application using particular framework.
- But usage of the particular framework as a main driving force for application architecture is much like the tail wagging the dog.

Given how hype-driven the software development industry these days, you may be sure that in a couple of years new frameworks/libraries will ride the edge of frontend development.
As soon as the framework of today's choice goes out of the trend - you will be either maintaining the "legacy" codebase or starting the re-write of the whole UI application.
Both options are sort of stealing from the customer. Re-writing of the application due to framework change doesn't introduce any benefits to business. Maintainance of the "legacy" codebase drastically reduces the motivation of the team and subsequently - individual developers performance.

This article shows how the UI could be built based on the high-level design patterns. Concrete frameworks/libraries are chosen only in order to cover responsibilities, defined by architecture.

### 2. Architectural goals and limitations

#### Goals:
1. New developer can take a brief look on the code structure and get an intent of application
2. Force developers to sepation of concerns and thereby force the code to be modular so:
  1. When we do some nasty hacking OR want to integrate with external boundary - code does not spill through multiple files and therefore it's replacement become a realistic task rather then "abstract long-term refactoring".
  2. Modules are testable
3. Force logically related things to be located close to each other in the folders structure and avoid the need to search for very related code in very distant folders. Check the <a href='https://martinfowler.com/bliki/PresentationDomainDataLayering.html'>article</a> covering the differences between the code separation by *technical tier* rather than by *functional responsibility*.
4. Account for loose coupling between modules so changes to implementation details of particular module (or replacement of this module) do not affect other modules as long as interfaces are met
5. Mechanics chosen for modules integration does not introduce inaccepatble permormance issues.
6. Dependencies on particular libraries are not spilling through the whole codebase, but being manageable within a limited amount of codebase, responsible for integration with such a libraries/frameworks.

#### Limitations:
Application should run in browser. Subsequently it should be written (or compiled into) the HTML+CSS for static UI and JavaScript for adding dynamic bahaviors to this look.

### 3. Narrow down the article scope:

There are a plenty of high-level approaches to code structuring. The most noticible are **layered**, **onion** and **hexagonal**.

This article is scoped to reflect only the presentation layer of layered/onion because the wast majority of the SPA are primarily focused on *showing* the data. 
So these applications could simply bypass *Domain* layer entirely and have a tiny *Application* layer (or bypass this layer as well). 

Subsequently, the most convenient way to understand the intent and the purpose of such an application is to get an overall picture of the presentation-related code.

Hovewer, this article keeps presence of the other layers in mind and describes mechanics for using the *domain-* and *application-*level functions/classes if your application have such a layer(s).

Notice that if both layers are bypassed you end up in classic *Hexagonal* (so called <a href='https://alistair.cockburn.us/hexagonal-architecture/'>Ports and Adapters</a>) approach where your presention IS your application.
Check out how the integration with local storage is extracted to the **boundaries/local-storage** folder within the <a href='https://dgrudzynskyi.github.io/todomvc-mobx-react-mvvm/'>TodoMVC sample</a>.

### 4. Files structure. How to make the SPA <a href='https://blog.cleancoder.com/uncle-bob/2011/09/30/Screaming-Architecture.html'>scream</a>? 
Let's take a look on the typical online shop. This is how it would be likely drawn on a nakpin by a customer:

<div class="block-with-image-container block-with-image-container--narrow">
  <div class='image-with-legend-container'>
    <img 
        src="{{site.baseurl}}/assets/2021-03-05/napkin-store.png" 
        alt="Typical online shop structure, drawn on a napkin"
    />
    <br />
    <span class='image-legend'>Figure 1: typical online shop structure, drawn on a napkin</span>
  </div>

  <div class='image-with-legend-container image-with-legend-container--small image-with-legend-container--left'>
    <img 
        src="{{site.baseurl}}/assets/2021-03-05/structure-with-pages.png"
        alt="top level presentation codebase structure" 
    />  
    <span class='image-legend'>Figure 2: top level folders structure, which reflects logical structure from figure 1</span>
  </div>
  <p>
    What could be the most screaming way to structure the codebase? Check the figure 2. All top-level blocks are reflected as separate folders.
  </p>
  <p>
    Notice that we haven't forgotten to add a 'shared' folder in order to reflect all 'shared' UI blocks, such as layout, navigation bar, basket.
  </p>

  <div class='image-with-legend-container image-with-legend-container--small image-with-legend-container--right'>
    <img 
      src="{{site.baseurl}}/assets/2021-03-05/structure-with-pages-parts.png"
      alt="placing page sub-parts within the 'parts' folder" 
    />  
  <span class='image-legend'>Figure 3: placing page sub-parts within the 'parts' folder</span>
  </div>
  <p>
    Our pages are built from the logical (and visual) parts. By now let's call them 'parts' and drop in the folder with same name. Take a   look on what we've got on the figure 3.
  </p>
  <p>
    Seems that nesting becomes annoying once we have the second level of hierarchy in 'goods catalogue' page. Path <i>goods-catalogue/parts/goods-list/parts/good-details.js</i> is already on the edge of reasonable path lengh. While this is not the deepest hierarchy we'd potentially have in the real application.
  </p>
  
  <div class='image-with-legend-container image-with-legend-container--small image-with-legend-container--left'>
  <img 
    src="{{site.baseurl}}/assets/2021-03-05/structure-with-pages-no-parts.png"
    alt="hoist page sub-parts from the 'parts' folders" 
  />  
  <span class='image-legend'>Figure 4: hoist the sub-parts from the 'parts' folders</span>
  </div>
  <p>
    Now, let's get rid of unnecesary "parts" leaf in folders structure. Check the figure 4.
  </p>
  <p>
    Now inside the <i>goods-catalogue/goods-list</i> we have three files. <b>goods-list.js</b> (parent one) -  is located in between of the files, used for the definition of the sub-parts. On the real-scale projects, giving the presence of multiple files (styles, js, html, you name it) this will results in a mess an inability to distinguish between the <i>part</i> and it's <i>sub-parts</i>. 
  </p>
  
  <div class='image-with-legend-container image-with-legend-container--small image-with-legend-container--right'>
    <img 
        src="{{site.baseurl}}/assets/2021-03-05/structure-components2.png"
        alt="application of the underscore to distinguish between the part and it's subparts" 
    />  
    <span class='image-legend'>Figure 5: application of the underscore to distinguish between the part and it's subparts</span>
  </div>
  <p>
    The solution:
    <ol>
      <li>
        If particular part is composed from multiple files - create a folder for it. 
        <ul>
         <li><b>goods-list</b> is a part, and it consists from more than one file, so it have a dedicated folde. </li>
         <li><b>Filters</b> is the part, consists from single file - so drop this file without creation of separate folder for it. </li>
        </ul>
      </li>
      <li>
        If particular part (either one-file part or multiple-files part) is a <i>sub-part</i> - prefix it with the underscore <i>"_"</i> sign. This way, all <i>sub-parts</i> (child items) are pinned to the top of the folder in the files explorer.
        <ul>
            <li><b>goods-list folder</b> is a sub-part of <b>goods-catalogue</b> so it is prefixed with underscore.</li>
            <li><b>_goods-list.js</b> relates directly to the folder it locates within, so it is not prefixed with underscore.</li>
            <li><b>_good-details.js</b> is a sub-part of <b>goods-list</b> so it is not prefixed with underscore.</li>
        </ul>
      </li>
    </ol>
  </p>
</div>

<p>
Done! Once you get used to this approach, a brief look on every folder will let you instantly identify the <i>sub-parts</i> and open the file, used to assemble all these <i>sub-parts</i> together.
Notice, that folder <b>pages</b> was renamed into the <b>components</b> on the last screenshot. It is done because <i>pages</i> and <i>page parts</i> are technically different things, but in html terms both can be called a <a href='https://developer.mozilla.org/en-US/docs/Web/Web_Components'>component</a>. So from here onvards, folder <i>components</i> become the main folder of our application and become a "home" for the presentation layer of SPA.
</p>


### 5. Programming language. JavaScript?

The only language which can be executed in the browser is JavaScript.
There are a lot of articles describing how cumbersome it is. You can <a href='https://www.destroyallsoftware.com/talks/wat'>laught about it (timecode 1-20)</a>, but it is just a funny part...

What's important to notice is that new features are being constantly added to JavaScript. The <a href='https://tc39.es/ecma262/'>Specification</a> is being updated every year. All new features comes through the <a href='https://github.com/hemanth/es-next'>4-stages review process</a> before being pulled into the specification. But many times they are being adopted by browsers "before" reaching stage 4. And many times community and framework authors starts using features before they are being included into the specification. For example, <a href='https://github.com/tc39/proposal-decorators'>decorators</a> started being widely used in 2015, while still not being included in the specification. In the other hand, often times business requires an application to works in old browsers, lacking the support of newly introduced features.

Therefore even if you are using plain JavaScript, you have to use a transpiler (babel) to make a "browser-compatible" JavaScript from the "wild-and-modern" JavaScript.
Since the use of the transpiler is inevitable - there is no reason to restrict yourself from using another, more safe, more fancy and more useful language and compile it to JavaScript instead of using plain Javascript.

Deep analisis of available options is out of the scope of this article, but my personal choice is the <a href='https://www.typescriptlang.org/'>TypeScript</a> because:
- Provides compile-time types checking
- Being a superset of JS, it can execute imported JS functions as a part of it's own codebase without any additional integration code.
- Type definitions (typings) can be added on top of exisiting JavaScript codebase without affecting this codebase. Given the siplicity of adding the typings, majority of npm packages already have them and therefore almost every single third-party JavaScript library could be consumed by your application as TypeScript library. So most of the time integration with external libraries is also type-safe.

<span class='hint'>Hint: i'd suggest to take a look into <a href='http://asmjs.org/'>asm.js</a>, <a href='https://docs.microsoft.com/en-us/aspnet/core/blazor/?view=aspnetcore-5.0'>blazor</a> and <a href='https://elm-lang.org/'>elm</a> if you're interested in other options</span>

### 6. Software design targets:

Let's remember the limitations, applied by the browsers: HTML, CSS, JavaScript.
Remember the file structure we want to have (chapter 4) - foldres tree which mirrors visual elements tree.

So the **first target [6.1]** for the presentation layer structure is to let the components being defined using HTML and CSS and then re-used.

One huge disadvantage of pure HTML is that it is not strictly typed. There are a plenty of templating engines, such as <a href='https://underscorejs.org/#template'>underscore.js</a>, <a href='https://handlebarsjs.com/'>handlebars.js</a> but they are designed to consume pure strings as an input. This prevents us from compile-time check on templates validity.

So the **second target [6.2]** is to ensure that we can define TypeScript interfaces to reflect all properties, used within the component. And then during compile-time throw an error if an unknown property is accessed within the component or an unknown attribute is defined on component in the html markup.

Every UI element on the page might have different states based on *some data*"*. Native HTML elements receive this data as <a href='https://www.w3schools.com/html/html_attributes.asp'>html attributes</a>. This is enough for static markup. For dynamic markup we want to have the data storages with values, being changed while user is working with application.
Same time, we shouldn't lose an ability to pass the data to components through the attributes.

So the **third target [6.3]** is to let the components to consume data from both attributes and data stores. Let components being re-rendered when data is changed.

And the **fourth target [6.4]** - define data stores specification:
- let the data stores be shared between different components in order to reuse single source of truth between multiple tiny UI components reflecting "parts" of the dataset.
- let the data stores be created per-component in order to support scenarios where different components have separate datasets behind them.
- let the data stores use domain and/or application services. In order to avoid tough coupling between the presentation layer and application boundaries, services shall be consumed using <a href='https://www.infoq.com/articles/Succeeding-Dependency-Injection/'>Dependency Injection</a>. Data stores should only rely on the interfaces. 


Finally, we don't want the data in data stores being public so it can not be accidentally changed during the component's rendering or within the event handler, defined in the component. We want to keep components as dumb as possible and make them nothing complex than the strongly-typed-and-optimized-html-templates. In order to achieve this, let's keep the state of data stored encapsulated within this stores, exposing the methods for access/modification of such a state. In other words, let's define our stores in form of old-fashioned classes.

Hovewer, as i've mentioned above, the data store might be shared as single source of truth by multiple tiny components.
In this case we want to have a very explicit visibility of the data slice, consumed by concrete component. This serve: 
1. Code readability (developer can guess the component purpose by seen data it receive).
2. Performance (re-renders of the component could be avoided if data, not related to the effective data slice, is changed).

So the **fifth target [6.5]** - let the data stores be defined as TypeScript classes, define mechanics to identify the slice of data, required by particular component.

With these targets in mind, let's define following logical pieces code units:
- components - strongly typed html template + css stylesheet 
- viewmodels - classes, encapsulating the state of the hierarchy down the component and exposing the methods for access/modification of such a state.
- viewmodel facades - limit the visibility of viewmodel properties to ones, required by particular component.

<div class="block-with-image-container block-with-image-container--medium">
  <div class='image-with-legend-container'>
    <img 
      src="{{site.baseurl}}/assets/2021-03-05/intended-code-diagram.png" 
      alt="desired design of presentation layer"
    />
    <br />
    <span class='image-legend'>Figure 6: desired design of presentation layer</span>
  </div>
  <ul>
    <li>solid arrows reflects components being rendered by parent component, with attributes set supplied directly in the markup. </li>
    <li>dashed arrows reflects code units being referenced by another code units.</li>
    <li>blocks with green border - module boundaries. Each module/submodule is represented by dedicated folder. Shared modules lay within the "shared" folder.</li>
    <li>blue blocks - viewmodels. viewmodels are defined "per module/submodule".</li>
  </ul>
</div>

<div class="block-with-image-container block-with-image-container--medium">
  <div class='image-with-legend-container image-with-legend-container--right' style='width:400px'>
    <img 
      src="{{site.baseurl}}/assets/2021-03-05/intended-code-diagram-widget-attrs.png" 
      alt="passing attributes to viewmodel"
    />
    <br />
    <span class='image-legend'>Figure 7: attributes should be passed not only to the root component of module, but also to it's viewmodel</span>
  </div>
  <p>
    Does we miss anything? Notice that the viewmodels on the figure 6 does not have any parameters. This is always true for the "top-level" or "global" viewmodels. But submodule viewmodels, as well as shared modules viewmodels may frequently rely on the parameters, defined during the application execution.
  </p>
  <p>
    So the <b>sixth target [6.6]</b> - let the attributes of the sub-module to be consumed by the viewmodel of this sub-module.
  </p>
</div>

### 7. Technical choices:

I am going to use mainstream libraries in order to make this article easier for read. Hence there will not be detailed comparison of different options - more popular library will be used.

#### 7.1. Components:

We want to have a library, which would allow the strongly-typed html markup definition.
This can be acheived by using tsx (typed <a href='https://reactjs.org/docs/introducing-jsx.html'>jsx</a>) syntax, supported by libraries such as <a href='https://reactjs.org/'>React</a>, <a href='https://preactjs.com/'>Preact</a> and <a href='https://infernojs.org/'>Inferno</a>.
Tsx is **!not!** the pure html, hovewer it can be automatically converted to/from pure html so we are fine if at some moment we decide to switch into 'pure' html.

I order to reduce dependency on a particular library, let's limit views to be a pure functions, receiving attributes and returning JSX node. This is the approach proven by/stolen from early-days react's <a href='https://reactjs.org/docs/components-and-props.html'>functional components</a>. 

<span class='hint'>hint: durign the last few years react components have become side-effect-full with introduction of react hooks. This is a brand new story, hooks should not be used while working with the approach, described in this article.</span>

In other terms, 'components' are stateless. Think about them through the simple equation `UI=F(S)` where 
- **UI** - visible markup
- **F** - component definition 
- **S** - state of current values within the viewmodel

Example component looks like this:

{% highlight jsx %}
interface ITodoItemAttributes {
  name: string;
  status: TodoStatus;
  toggleStatus: () => void;
  removeTodo: () => void;
}

const TodoItemDisconnected = (props: ITodoItemAttributes) => {
  const className = props.status === TodoStatus.Completed ? 'completed' : '';
  return (
    <li className={className}>
      <div className="view">
        <input className="toggle" type="checkbox" onChange={props.toggleStatus} checked={props.status === TodoStatus.Completed} />
        <label>{props.name}</label>
        <button className="destroy" onClick={props.removeTodo} />
      </div>
    </li>
  )
}
{% endhighlight %}

This component is responsible for rendering of the single todo item within the <a href='https://dgrudzynskyi.github.io/todomvc-mobx-react-mvvm/'>TodoMVC app</a>.

The only dependency we have in this code is the dependency on the JSX syntax. So this component can be rendered by any library, which can render JSX markup.
With this approach, replacement of particular library will still not come for free, but is manageable.

So we handle **targets [6.1] and [6.2]**. Also, we keep the code of particular feature library-agnostic as we do not referencing any concrete library in this code.

<span class='hint'>note: i am using react for referenced <a href='https://dgrudzynskyi.github.io/todomvc-mobx-react-mvvm/'>TodoMVC demo application</a>.</span>
 

#### 7.2. ViewModels 

As it was mentioned in the chapter [6], we want ViewModels to be written as classes in order to:
- Allow internal state encapsulation 
- Account for integration with domain/application layers using the dependency injection principle.

But classes do not have built in mechanics for automatic re-render of the components referencing the data, encapsulated by the concrete class instance.

We need something, called reactive UI. The comprehensive coverage of the principles can be found in <a href='https://github.com/meteor/docs/blob/version-NEXT/long-form/tracker-manual.md'>this doc</a>. This approach was initially introduced in WPF (C#) and was called <a href='https://docs.microsoft.com/en-us/archive/msdn-magazine/2009/february/patterns-wpf-apps-with-the-model-view-viewmodel-design-pattern#why-wpf-developers-love-mvvm'>Model-View-ViewModel</a>.
In the JavaScript world, objects served as observable data sources are mostly called <a href='https://mobx.js.org/defining-data-stores.html'>stores</a> following the <a href='https://facebook.github.io/flux/'>flux</a> terminology.
Hovewer, a **store** is a very generic term, it can define:
- Global data storage for the whole application
- Domain object, encapsulating domain logic, neither bound to particular component, nor shared across thw whole application.
- Local data storage for a concrete component or components hierarchy

So every viewmodel is a store, but not every store is viewmodel.

Couple more limitations we want to apply on viewmodels.
- Code, related to reactivity, should not be blended with the feature-specific code. 
- ViewModel shall not reference components and should now know that there are the compontens, referencing this viewmodel.

For now, i am going to us the <a href='https://mobx.js.org/README.html'>mobx</a> library and use decorators to make the regular class fields observable.
Take a look on the example viewmodel:

{% highlight typescript %}
class TodosVM {
    @mobx.observable
    private todoList: ITodoItem[];

    // use "pure man DI", but in the real applications there todoDao will be initialized by the call to IoC container 
    constructor(props: unknown, private readonly todoDao: ITodoDAO = new TodoDAO()) {
        this.todoList = [];
    }

    public initialize() {
        this.todoList = this.todoDao.getList();
    }

    @mobx.action
    public removeTodo = (id: number) => {
        const targetItemIndex = this.todoList.findIndex(x => x.id === id);
        this.todoList.splice(targetItemIndex, 1);
        this.todoDao.delete(id);
    }

    public getTodoItems = (filter?: TodoStatus) => {
        return this.todoList.filter(x => !filter || x.status === filter) as ReadonlyArray<Readonly<ITodoItem>>;
    }
	
	/// ... other methods such as creation and status toggling of todo items ...
}
{% endhighlight %}

Notice that we reference mobx directly, but decorators are never spilling through the method bodies.

I'll show you how to abstract out the reactivity and remove dependency on mobx in the next article. For now it is enough to just prefix the decorators with the 'mobx' namespace. The moment developer might decide to change the reactivity providing library  - namespace might be replaced with the another one using automated script.

Also notice that ViewModel receive the first argument of type `unknown` in the constructor. This accounts to the **target [6.6]**. Given that `TodosVM` does not depends on any attributes, the type of props is `unknown`. But in order to generalize the ViewModel interface, its constructor shall always receive the first argument of the type matching an attributes of the root component of the (sub-)module:

{% highlight typescript %}
interface IVMConstructor<TProps, TVM extends IViewModel<TProps>> {
    new (props: TProps, ...dependencies: any[]) : TVM;
}

interface IViewModel<IProps = Record<string, unknown>> {
    initialize?: () => Promise<void> | void;
    cleanup?: () => void;
    onPropsChanged?: (props: IProps) => void;
}
{% endhighlight %}

Notice that all methods of `IViewModel` are optional. They might be declared on the viewmodel in case we want to ensure that:
- Code is executed when viewmodel is created
- Code is executed when viewmodel is deleted. JavaScript does not define the "deletion" of the object and we can't do anything at the moment object is being garbage-collected. I use word "deleted" to describe the ViewModel object, which is no more used by any component instance.
- Code is executed when attributes of the (sub-)module is changed

Opposite to components, viewmodels are statefull. They are to be created once the module appears inside the DOM structure and deleted once module is removed from the DOM.
And as it is shown in the figure 7, top level component of the module is an "entry" of the module.
This drives us to the fact that viewmodel should be created when component instance is created(mounted) and deleted when it is deleted(unmounted). Let's handle this with the <a href='https://reactjs.org/docs/higher-order-components.html'>higher order components</a> technique.



Consider having the function of following signature:
{% highlight typescript %}
  type TWithViewModel = <TAttributes, TViewModelProps, TViewModel>
  (
    moduleRootComponent: Component<TAttributes & TViewModelProps>,
    vmConstructor: IVMConstructor<TAttributes, TViewModel>,
  ) => Component<TAttributes>
{% endhighlight %}

such a function shall return a higher order component over the `moduleRootComponent` which:
- must ensure that the viewmodel is created before the creation (and mounting) of the component.
- must ensure that viewmodel is cleaned up (aka deleted) when component is being unmounted from components tree.

You can check the <a href='https://github.com/DGrudzynskyi/todomvc-mobx-react-mvvm/blob/master/app/framework-extensions/with-vm.tsx'>implementation</a> used for referenced <a href='https://dgrudzynskyi.github.io/todomvc-mobx-react-mvvm/'>TodoMVC app</a>.
This implementation is a bit more complex than described one in order to handle custom IoC container and re-creation of the viewmodel in responce to attribute changes.

Now let's see the example of usage of this function:

{% highlight jsx %}
const TodoMVCDisconnected = (props: { status: TodoStatus }) => {
    return <section className="todoapp">
        <Header />
        <TodoList status={props.status} />
        <Footer selectedStatus={props.status} />
    </section>
};

const TodoMVC = withVM(TodoMVCDisconnected, TodosVM);
{% endhighlight %}

Within the markup of application entry page (or root router, whatever you prefere) it is to be called as `<TodoMVC status={statusReceivedFromRouteParameters} />`
After that, instance of `TodosVM` becomes accessible by components down the component hierarchy.

Notice that `withVM` hides the implementaion details of the viewmodel creation and passing down the components hierarchy.  

- `TodoMVCDisconnected` component is library-agnostic
- `TodoMVC` component can be rendered within the library-agnostic component
- `TodosVM` reference only mobx decorators, and it is something which could be easily abstracted out but not in scope of this article

<span class='hint'>note: in the provided implementation, `withVM` function rely on the react context API, but you may experiment to find the better-faster-stronger way to implement it.
What's important is that implementation shall be done in conjunction with the implementation of `connectFn` (mentioned in the next chapter) in order to ensure that the same viewmodel is reused across all components within the components hierarchy. </span>
 
#### 7.3. ViewModel facades.

<div class="block-with-image-container block-with-image-container--medium">
  <div class='image-with-legend-container image-with-legend-container--right' style='width:400px'>
    <img 
      src="{{site.baseurl}}/assets/2021-03-05/intended-code-diagram-widget-attrs-and-connect.png" 
      alt="passing the component's attributes to the slicing function"
    />
    <br />
    <span class='image-legend'>Figure 8: passing the component's attributes 'IWidgetAttrs2' to the viewmodel facade (aka slicing function)</span>
  </div>
  <p>
    Take a look on the figure 6 within the chapter 6. <a href='https://en.wikipedia.org/wiki/Facade_pattern'>Facade</a> define the class, exposing the subset of the wrapped module instead of its complete interface.
    Hovewer, creation of the additional classes to wrap the ViewModel is quite verbose.
  </p>
  <p>
    Instead of classic "facades" let's use functions, which would take a ViewModel (or multiple viewmodels) as an argument(s), and return the data slice of interest.
    Let's call it <b>slicing function</b>
    What if such a function receives the attributes of the component, as its last argument? With single ViewModel in mind, function signature will looks like:
  </p>
</div>

{% highlight typescript %}
type TViewModelFacade = <TViewModel, TOwnProps, TVMProps>(vm: TViewModel, ownProps?: TOwnProps) => TVMProps
{% endhighlight %}

Looks very similar to the Redux's <a href='https://react-redux.js.org/api/connect'>connect function</a>. But instead of `mapStateToProps`, `mapDispatchToActions` and `mergeProps` we have single function to return both 'state' and 'actions' from the viewmodel. 
Here is the slicing function from provided TodoMVC example: 

{% highlight typescript %}
const sliceTodosVMProps = (vm: TodosVM, ownProps: {id: string, name: string, status: TodoStatus; }) => {
    return {
        toggleStatus: () => vm.toggleStatus(ownProps.id),
        removeTodo: () => vm.removeTodo(ownProps.id),
    }
}
{% endhighlight %}

<span class='hint'>notice: i am calling component's attributes as 'OwnProps' to align it with terminology, widely spread in the react/redux world.</i>

This similarity drives us to the most convenient way of utilizing such a slicing functions within the components tree - using the higher order components.
Assume viewmodel is already hosted up the components hierarhy by using HOC, generated by `withVM` function.
Let's define a signature of a function, which receive slicing function and the component and return higher-order component, bound to such a ViewModel.

{% highlight typescript %}
type connectFn = <TViewModel, TVMProps, TOwnProps = {}>
(
    ComponentToConnect: Component<TVMProps & TOwnProps>,
    mapVMToProps: TViewModelFacade<TViewModel, TOwnProps, TVMProps>,
) => Component<TOwnProps>

const TodoItem = connectFn(TodoItemDisconnected, sliceTodosVMProps);
{% endhighlight %}

Rendering of such an item within the items list: `<TodoItem id={itemId} name={itemName} status={itemStatue} />`

Notice that `connectFn` hides the implementaion details of the reactivity. 
- It takes the component `TodoItemDisconnected` and slicing function `sliceTodosVMProps` - both unaware of the reactivity library and JSX rendering library. 
- It returns the component, which will be re-rendered reactively once the data, encapsulated by ViewModel, is modified.

You can check the actual implementation of  <a href='https://github.com/DGrudzynskyi/todomvc-mobx-react-mvvm/blob/master/app/framework-extensions/create-connect.tsx'>connectFn in the</a> TodoMVC app.

### 8. Conslusion:

What is the most intricated part part about the designed code structure?
All features-related code is written using the framework-agnostic syntax. 
Typescript objects, typescript functions, TSX - that's all we are bound to.

Guess that after reading this article you might see the benefits of working on SPA architecture ahead of development start. I wish this article would change the mindset from *"pick the hyped framework, get the shit done"* to *"analise what should be done and define a proper toolset"* even for UI applications.

#### But could the whole presentation layer be framework-agnostic inside the production-grade application?
In order to remove references to mobx, react and mobx-react from presentation layer, we should do slightly more:
- Abstract out mobx decorators
- Abstract out every framework-dependant library, used by the presentation layer.
For example rovided <a href='https://dgrudzynskyi.github.io/todomvc-mobx-react-mvvm/'>TodoMVC sample</a> rely on the react-router/react-router-dom libraries.
- Abstract out the synthetic events signatures, specific to particular JSX rendering engine. 

First two items are easily manageable and I'll show you how to handle them in the further articles. 
Third one is kind of different beast. Such an abstraction means that we are going to build yet another framework, but without tests coverage, community support and all that things that matters.

I am pragmatically chosing to keep referencing react's synthetic events signatures in mine components.
Such approach makes swithing out of the react or of the mobx a manageable task. 
Even though switching our of the react will require some work, this work will be localized within the particular components and will not spill through to viewmodels or slicing functions.


### P.S. Comparison of the suggested structure and it's implementation to main popular development frameworks:

- Comparing to <a href='https://react-redux.js.org/'>React/Redux</a> combo: ViewModels replace *reducers*, *action creators* and *middlewares*. ViewModels ARE stateful. No time-travel. Multiple stores. No performance issues caused by excessive connect function usages with some logic inside. Redux-dirven app tends to become slower over time while more and more connected components are being added to application, so there is no evident bottleneck to fix. Slicing function implementation given in the example is not affected by this issue, cudos to <a href='https://mobx.js.org/understanding-reactivity.html'>mobx dependencies tracking</a>. Alhough, it can be acheived with another reactive UI library as well.

- Comparing to <a href='https://vuejs.org/'>vue</a>: Strongly typed views cudos to TSX. Viewmodels are native class instances without framework-specific syntax blended in. Vue.js force the definition of state to be done within <a href='https://vuejs.org/v2/api/#Options-Data'>the objects of specific structure</a> having 'data','methods', etc. properties.  Absence of vue-specific directives and model binding syntax.

- Comparing to <a href='https://angular.io/'>angular</a>: Strongly typed views cudos to TSX. Absence of angular-specific directives and model binding syntax within the html. Prevent views from direct manipulations on the state (aka on the component public fields, aka two-way data binging). Hint: for some scenarios (such as forms) two way data binding might be preferred.
 
- Comparing to pure react with state managed through hooks (such as <a href='https://reactjs.org/docs/hooks-state.html'>useState</a>/useContext): better separation of concerns. VMs could be threaten like the container components, hovewer they do not render anything and subsequently it is much harder to mess up the state management logic and UI code. 

  Avoid the neccessity to care about hooks sequence, about keeping the useEffects dependencies tracked in the 'deps' array, about keeping an information about whether the component is still mounted when asyn action is completed, about making sure closures from 'previous' renders are not suddenly used during effect execution. 

  As any other tech, hooks (and in particular - useEffect) requires developer to follow some recommendations (not declared in the interface, but accepted as a 'approach', 'mental model' or 'best practices'). There is a nice <a href='https://overreacted.io/a-complete-guide-to-useeffect/'>guide to useEffect</a> from one of react team members. Read it and then answer two questions: 
  - What do you acheive with hooks? 
  - How many rules, not controlled by compiler and hardly tracked by linter/visual code review, should be followed by developer while working with hooks in order to make them predictable? 
    Once the second list exceedes the first one by a huge margin - it is a good sign to me that an approach should be considered with a huge caution. <span class='hint'>hint: tiny bit of <a href='https://blog.logrocket.com/frustrations-with-react-hooks/'>frustration</a> about hooks</span>

- Comparing to <a href='https://mobx.js.org/react-integration.html'>react-mobx integration</a>.
Code structuring is not defined within the shipped in react-mobx docs. It is up to the development team. You may consider an approach, described in this article as an approach to structure the application written with react for rendering and mobx for state management.

- Comparing to <a href='https://mobx-state-tree.js.org/'>mobx-state-tree</a>: Viewmodels are native class instances without framework-specific syntax blended in. Mobx-state-tree <a href='https://mobx-state-tree.js.org/concepts/trees'>types definitions</a> heavily rely on the framework specific syntax. Mobx-state-tree may force the duplication of code while declaring the types because properties of particular type should be described both in the typescript interface and in the model definition.