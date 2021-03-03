Yet another take on SPA (single page application) frontent architecture. 

1) But... why?

There are a lot of frontend frameworks.
There are a lot of tutorials and documentation showing how to build an application using particular framework.
But usage of the particular framework as a main driving force for application architecture is much like the tail wagging the dog.
And given how hype-driven the software development these days, you may be sure that in a couple of years there will be some brand new tails.
As soon as the framework of today's choice goes out of the trend - you'll will be either maintaining the "legacy" codebase or starting the re-write of the whole UI application.
Both options are sort of stealing from the customer. Re-writing of the application due to framework change doesn't introduce any benefits to business. Maintainance of the "legacy" codebase drasticly reduces the motivation of the team and subsequently - individual developers performance.

This article shows how the UI could be built based on the high-level architectural patterns, with concrete frameworks/libraries chosen to cover responsibilities, defined by architecture.

2) Architectural goals and limitations

Good architecture stands for following needs:
 1. new developer can take a brief look on the code structure and get an intent of application
 2. force developers to follow sepation of concerns and thereby force the code to be modular so:
 2.1 when we do some nasty hacking OR want to integrate with external boundary - code does not spill through multiple files and therefore it's replacement become a realistic task rather then "abstract long-term refactoring".
 2.2 modules are testable
 3. Force logically related things to be located close to each other in the folders structure and avoid the need to search for very related code in very distant folders. Check the article https://martinfowler.com/bliki/PresentationDomainDataLayering.html for details, which is quite explanatory on this topic.
 4. account for loose coupling between modules so changes to implementation details of particular module (or replacement of this module) do not affect other modules as long as interfaces are met
 5. mechanics chosen for modules integration does not introduce inaccepatble permormance issues.

Limitations to architecture:
Application should run in browser. Subsequently it should be written (or compiled into) the HTML+CSS for static look and JavaScript for adding dynamic bahaviors to this look.

3) Narrow down the article scope:

There are a plenty of high-level architectural approaches to code structuring. The most noticible are "layered", "onion" and "hexagonal".

This article is scoped to reflect only the presentation layer of layered/onion because the wast majority of the SPA are primarily focused on "showing" the data. 
So these applications could simply bypass "Domain" layer entirely and have a tiny "Application" layer (or bypass this layer as well). 

Subsequently, the most convenient way to understand the intent and the purpose of such an application is to get an overall picture of the presentation-related code.

Hovewer, this article keeps presence of other layers in mind and describe mechanics for using the domain- and application-level functions/classes if your application have such a layer(s).

Notice that if both layers are bypassed you end up in classic "Hexagonal" (so called "Ports and Adapters") approach where your presention IS your application. (which is true for majority of front-end applications). !!!!!In this article i'll give an example of defining an outgoing port and plugging in an external service adapter.!!!!!


4) Files structure. How to make the SPA scream? (https://blog.cleancoder.com/uncle-bob/2011/09/30/Screaming-Architecture.html)

Let's take a look on how the typical online shop would be likely drawn on a nakpin by a customer:

[pic] - schema

What could be the most screaming way to structure the codebase? Take a look below:

[pic] - top level files structure

Of course, we haven't forgotten to add a 'shared' folder in order to reflect all 'shared' UI pieces, such as layout, navigation bar, basket.
Our pages are built from the logical (and visual) parts. By now let's call them 'parts' and drop in the folder with same name. Take a look on what we've got:

[pic] - expanded 'checkout' 'goods catalogue'

You may notice that the nesting becomes quite annoying once we have the second level of hyerarchy in 'goods catalogue' page.
So the taks number 1 is to get rid of unnecesary "parts" leaf in folders structure. Let's do it for 'goods catalogue':

[pic] - with pages without parts

Now inside the goods-catalogue/goods-list we have three files. And the "parent" one - "goods-list.js" is located between the "child" ones. On the real-scale projects, where you might build a single part of multiple files (styles, js, html, you name it) these will be a total mess where you'll not be able to tell the difference between parent and a child from until taking a look on an every file individually. 
The solution: 
1. If particular part is composed from multiple files - create a folder for it. Goods-list is a part, and it consists from more-than-one file. So we have a dedicated folder for it.
2. If particular part (either one-file part or multiple-files part) is a "child" - prefix it with the underscore "_" sign. This way, all "child" items are pinned to the top of the folder, while the files, related to the folder subject are all located below them:

[pic] - structure components.

Done! Once you get used to this approach, a brief look on every folder will let you instantly identify the "nested sub-parts" and open the file, used to assemble all these sub-parts together.
Notice, that folder "pages" was renamed into the "components" on the last screenshot. It is done because "pages" and "page parts" are technically different things, but in html terms both can be called a "component". So from here onvards, folder "components" become the main folder of our application and become a "home" for the presentation layer of SPA.

5) Programming language. JavaScript?

The only language which can be executed in the browser is JavaScript.
There are a lot of article describing how cumbersome it is. You can laught about it a bit watching https://www.destroyallsoftware.com/talks/wat (from 1-20), but it is just a funny part...

What's important to notice is that new features are being constantly added to JavaScript. Specification is being updated every year (see https://tc39.es/ecma262/). New features comes through the 4-stages review process (https://github.com/hemanth/es-next) before being pulled into the specification. But many times they are being adopted by browsers "before" reaching stage 4. And many times community and framework authors starts using features before they are being included into specification. For example, decorators started being widely used in 2015, while still not being included in the specification. In the other hand, often times business requires an application to works in old browsers, lacking the support of newly introduced features.

Therefore even if you are using plain javascript, you have to use a transpiler (babel) to make a "browser-compatible" javascript from the "wild-and-modern" javascript.
Hence the use of the the transpiler is mandatory - there is no reason to restrict yourself from using another, more safe, more fancy and more useful language and compile it to JavaScript instead of using plain Javascript.

Deep analisis of available options is out of the scope of this article, but my personal choice is the TypeScript because:
- provides compile-time types checking
- being a superset of JS, it can execute imported JS functions as a part of it's own codebase without any additional integration code.
- type definitions (typings) can be added on top of exisiting JavaScript codebase without affecting this codebase. Given the siplicity of adding the typings, majority of npm packages already have them and therefore almost every single third-party JavaScript library could be consumed by your application as TypeScript library. So most of the time integration with external libraries is also type-safe.

Hint: i'd suggest to take a look into asm.js, blazor and elm if you're interested in another options

6) Presentation layer code structure:

Let's remember the limitations, applied by the browsers: HTML, CSS, javascript.
Remember the file structure we want to have (chapter 4) - foldres tree, mirroring visual elements tree.

There is a concept of "Component" existing in HTML world https://developer.mozilla.org/en-US/docs/Web/Web_Components . I am not going to fit the application into spec-compatible web-components due to their verbosity. Hovewer let's consider every custom visual UI element as just a "component".

So the first goal for the presentation layer structure is to let the components being defined using HTML and CSS and then re-used.

One huge disadvantage of pure HTML is that it is not strictly typed. There are a plenty of templating engines, such as underscore.js, handlebars.js but they are designed to consume pure strings as an input. This prevents us from compile-time check on templates validity.

So the second goal is to ensure that we can define typescript interfaces to reflect all properties, used within the component. And then during compile-time throw an error if: unknown properties is accessed within the component or unknown attribute (not matching any properties name) is defined on component in the html markup.

Every UI element on the page might have different states based on "some data". Native HTML elements receive this data as html attributes. This is enough for static markup. For dynamic markup we want to have the data storages with values, being changed while user is working with application.
Same time, we shouldn't lose an ability to pass the data to components through the attributes.

So the third goal is to let the components to consume data from both attributes and data stores. Let components being re-rendered when data is changed.

And the fourth goal - let the data stores be defined:
- let the data stores be shared between different components in order to reuse single source of truth between multiple tiny UI components reflecting "parts" of the dataset.
- let the data stores be created per-component in order to support scenarios where different components have separate datasets behind them.
- let the data stores use domain and/or application services. In order to avoid tough coupling between the presentation layer and application boundaries, services shall be consumed using Dependency Injection. Data stores should only rely on the interfaces. 


Finally, we don't want the data in data stores being public so it can not be accidentally changed during the component's rendering or within the event handler, defined in the component. We want to keep components as dumb as possible and make them nothing complex than the strongly-typed-and-optimized-html-templates. In order to achieve this, let's follow the OOP principles and keep the state of data stores encapsulated within this stores, exposing the methods for access/modification of such a state.

Hovewer, as i've mentioned above, the data store might be shared as single source of truth by multiple tiny components.
In this case we want to have a very explicit visibility of the data slice, consumed by concrete component. This serve: 
1) Code readability (developer can guess the component purpose by seen data it receive).
2) Performance (re-renders of the component could be avoided if data, not related to the effective data slice, is changed).

So the fifth goal - let the data stores be defined as typescript classes, define mechanics to identify the slice of data, required by particular component.


With these goals in mind, let's define following logical pieces code units:
- components - strongly typed html template + css stylesheet 
- viewmodels - data stores, encapsulating the state of some component's hierarchy and exposing the methods for access/modification of such a state.
- viewmodel facades - account for picking only properties required by particular component from the viewmodel. So the 'whole' viewmodel is not being passed to particular component.

Here is how such structure could be visually reflected: 
[intended-code-diagram.png]

solid arrows reflects components being rendered by parent component, with attributes set supplied directly in the markup. 
dashed arrows reflects code units being referenced by another code units.
blocks with green border - module boundaries. Each module/submodule is represented by dedicated folder. Shared modules lay within the "shared" folder.
blue blocks - viewmodels. viewmodels are defined "per module/submodule".

Does we miss anything? Notice that the viewmodels does not have any parameters. This is always true for the "top-level" or "global" viewmodels. But submodule viewmodels, as well as shared modules viewmodels may frequently rely on the parameters, defined during the application execution.

so the sixth goal - let the attributes of the sub-module be consumed by the viewmodel of this sub-module.
for sub-module it is going to looks like 
[intended-code-diagram-widget-attrs.png]

7) Technical choices:

I am going to use mainstream libraries in order to make this article easier for read. 

7.1) Components:

We want to have a library, which would allow the strongly-typed html markup definition.
This can be acheived by using tsx (typed jsx) syntax, supported by libraries such as React, Preact and Inferno.
JSX is !not! the pure html, hovewer it can be automatically converted to/from pure html so we are fine if at some moment we decide to switch into 'pure' html.

I order to reduce dependency on a particular library, let's limit views to be a pure functions, receiving attributes and returning JSX node. This is the concept taken from early-days React functional components approach (before they become side-effect-full with react hooks).

In other terms, 'components' are stateless. The can be reflected by equation: UI=F(S) where 
*UI* - visible markup, 
*F* - component definition 
*S* - state of current values within the viewmodel

Example component would looks like this:

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

This component is responsible for rendering of the single todo item within the standard TodoMVC app. 
Notice the standard "presentational" component in "react" slang.

The only dependency we have in this code is the dependency on the JSX syntax. So this component can be rendered by any library, which can render JSX markup.
With this approach, replacement of particular library will still not come for free, but is manageable.

So we handle [first] and [second] goals, defined in section [6]. Also, we keep the code of particular feature library-agnostic as we do not referencing any concrete library in this code.

note: i am using React for referenced TodoMVC demo [link].
 

7.2) ViewModels 

As it was mentioned in the chapter [6], we want ViewModels to be written as classes in order to:
- allow internal state encapsulation 
- account for integration with domain/application layers using the dependency injection principle.

What's going beyond native javascipt syntax is an ability to automatically re-render components in respond to changes in data, encapsulated by viewmodels.

note: This "reactive" UI approach was initially introduced in WPF (C#) where implementation of INotifyPropertyChanged interface by viewmodel caused the re-render of particular pieces in the markup (xaml). This is when term MVVM first came to the market and this is why I found usefull to call the data stores 'viewmodels' even though they are oftenly called just 'stores' in FE community.

note x2: The comprehensive coverage of "reactive UI" principles given in javascript language can be found by link https://github.com/meteor/docs/blob/version-NEXT/long-form/tracker-manual.md

So we want to:
- ensure that any change to viewmodel field is reflected by all components referencing this field
- avoid blending the code, related to reactivity, with the feature-specific code. Viewmodel methods shall know nothing about the fact that there are some components using the data, changed by such methods. 

For now, i am going to us the "mobx" library and use decorators to make the regular class fields observable.
Take a look on the example viewmodel:

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
	
	/// ... other methods such are removal and status toggling on todo items ...
}

Notice that we reference mobx directly, but decorators are never spilling through the method bodies.

I'll show you how to abstract out the mobx in the next article. For now it is enough to just prefix the decorators with the 'mobx' namespace.
The moment developer might decide to change the library providing reactivity - namespace might be replaced with the another one using automated script while all the logic remains untouched.  

Also notice that we receive the first argument of type 'unknown' inside the ViewModel constructor. This accounts to the [sixth] goal, defined in the section [6]. Hence this ViewModel does not depends on any attributes, the type of props is 'unknown'. But in order to generalize the ViewModel interface, its constructor shall always receive the first argument having the type matching an attributes of the root component of the (sub-)module:

interface IVMConstructor<TProps, TVM extends IViewModel<TProps>> {
    new (props: TProps, ...dependencies: any[]) : TVM;
}

notice that we rely on IViewModel interface here:
interface IViewModel<IProps = Record<string, unknown>> {
    initialize?: () => Promise<void> | void;
    cleanup?: () => void;
    onPropsChanged?: (props: IProps) => void;
}

Notice that all methods are optional. They might be declared on the viewmodel in case we want to ensure that:
- Code is executed when viewmodel is created
- Code is executed when viewmodel is deleted (notice that javascript does not define the "deletion" of the object). I use word "deleted" for viewmodel object, which is no more used by any component
- Code is executed when attributes of the (sub-)module is changed

Opposite to components, viewmodels are statefull. They are to be created once the module appears inside the DOM structure and deleted once module is removed from the DOM.
And as it is shown in the figure [X], top level component of the module is an "entry" of the module.
This drives us to the fact that viewmodel should be created when component instance is created(mounted) and deleted when it is deleted(unmounted).

Let's handle this with the higher order components technique.
Consider having the function of following signature:

type TWithViewModel = <TAttributes, TViewModelProps, TViewModel>
	(
		moduleRootComponent: Component<TAttributes & TViewModelProps>,
		vmConstructor: IVMConstructor<TAttributes, TViewModel>,
	) => Component<TAttributes>

such a function shall return a higher order component over the moduleRootComponent, which:
- must ensure that the viewmodel is created before the creation (and mounting) of the component.
- must ensure that viewmodel is cleaned up (aka deleted) when component is being unmounted from components tree.

You can check the implementation in the repository [link to repository]:
Although it is a bit more complex to account for plugging in a custom IoC container and re-creation of the viewmodel in responce to changes in attributes.

Consider a following root component:
const TodoMVCDisconnected = (props: { status: TodoStatus }) => {
    return <section className="todoapp">
        <Header />
        <TodoList status={props.status} />
        <Footer selectedStatus={props.status} />
    </section>
};

such a function can be utilized in a following way: 

const TodoMVC = withVM(TodoMVCDisconnected, TodosVM);

then within the markup of application entry page (or root router, whatever you prefere) it is to be called as <TodoMVC status={statusReceivedFromRouteParameters} />

notice: code of the root module component and code of the viewmodel both not referencing any specific library.
All the references to mobx and to particular JSX rendering library (in or case it is react) are hidden within the withVM function.
In the provided implementation, "withVM" function rely on the react context API, but you may experiment to find the better-faster-stronger way to implement it.
What's important is that implementation shall be done in conjunction with the implementation of "connectFn" (mentioned in the next chapter) in order to ensure that the same viewmodel is reused across all components within the components hierarchy. 
 
7.3) ViewModel facades.

Take a look on the picture [x] within the chapter 6. Facade, as a pattern, define the class, exposing the subset functions of the wrapped module instead of it's complete interface.
Hovewer, creation of the additional classes to wrap the viewmodel is quite verbose.

What if instead of classic "facade" objects we use functions, which would take a viewmodel (or multiple viewmodels) as an argument(s), and return the data slice of interest?
What if whis such a function receive the attributes of the component, it is used for, as an argument as well?

show: intended-code-diagram-widget-attrs-and-connect.png

The signature of such a function can be defined as (with single viewmodel in mind so far): 

type TViewModelFacade = <TViewModel, TOwnProps, TVMProps>(vm: TViewModel, ownProps?: TOwnProps) => TVMProps

Have you identified something familiar in it?
Yep, it is very much similar to the Redux's "connect" function arguments. But instead of mapStateToProps, mapDispatchToActions and mergeProps we have single function slicing both 'state' and 'actions' from the viewmodel. 
Here is an example: 

const sliceTodosVMProps = (vm: TodosVM, ownProps: {id: string, name: string, status: TodoStatus; }) => {
    return {
        toggleStatus: () => vm.toggleStatus(ownProps.id),
        removeTodo: () => vm.removeTodo(ownProps.id),
    }
}

notice: i am calling component's attributes as 'OwnProps' to align it with terminology, widely spread in the react/redux world.

This similarity drives us to the most convenient way of utilizing such a slicing functions within the components tree - using the higher order components.
Let's define a signature of a function, which would receive slicing function and the component and return higher-order component, bound to viewmodel:

type connectFn = <TViewModel, TVMProps, TOwnProps = {}>
	(
        ComponentToConnect: Component<TVMProps & TOwnProps>,
        mapVMToProps: TViewModelFacade<TViewModel, TOwnProps, TVMProps>,
	) => Component<TOwnProps>

You can check the implementation in the repository [link to repository]:

Example of usage of this function within the TodoMVC application:


const TodoItem = connectFn(TodoItemDisconnected, sliceTodosVMProps);

then rendering of such an item within the items list JSX:

<TodoItem id={itemId} name={itemName} status={itemStatue}>

8) Conslusion:

What's the most intricated part part about the technical chioces made in chapter 7?
All features-related code is written using the framework-agnostic syntax. 
Typescript objects, typescript functions, JSX - that's all we lock on with this approach.

Could the whole presentation layer be framework - agnostic inside the production-grade application?
Yes, it could. Although, it will require an additional abstractions over the every framework-dependant library, used by the presentation layer.
For example provided [TodoMVC example] rely on the react-router/react-router-dom.
It will also require an abstraction over all JSX-rendering-library-specific syntax, such as synthetic events signatures and access to manual rendered DOM nodes.

Such an abstractions are not a subject for present article and might be discussed further on.
In my personal working experience routing 'have to' be abstracted out into some application-specific interfaces as there are usually a plenty of application-specific code. This might be the we want to reflect (or hide) URI parts, strongly typed routing and many more.
Same time abstraction over the react synthetic event system and 'ref' syntax seems like a major overengeneering for me.

All that said, current approach makes swithing out from react or from mobx a manageable task.
And the juicy part of it is that I've acheived it by architecting the structure "the way i want it to be" with clear separation of concerns and screaming files structure, rather then by trying to fit 'the bells and whistles' of 'libraryX' within the 'frameworkY'.

I hope this article shows you the importance of software architecture.
I also hope that next time you're going to start the development of new application (event UI one!), such a development will be started from analysing the "desired structure" rather than from picking the random hyped tool and using it for any case.

9) P.S.

Comparison of suggested structure to main popular development frameworks:


Comparing to React/Redux combo: ViewModels replace reducers/action creators/middlewares. It IS stateful. No time-travel. Multiple stores. No performance issues caused by excessive connect function usages with some logic inside. Redux-dirven app tends to become slower over time with an addition of more and more connected components (so there is no evident bottleneck to fix!). ViewModel facades (connect functions) implementation given in the example does not affected by this issue, cudos to mobx dependencies tracking (but it can be acheived with implementation, based on another reactive UI library.

Comparing to vue: Viewmodels are native class instances without framework-specific syntax blended in. Vue.js force the definition of state to be done within the objects of specific structure having 'data','methods', etc. properties. Strongly typed views cudos to TSX. Absence of vue-specific directives and model binding syntax.

Comparing to angular: strongly typed views. Strongly typed views cudos to TSX. Absence of angular-specific directives and model binding syntax. Prevent views from direct manipulations on the state (aka on the component public fields, aka two-way data binging). Hint: for some scenarios (such as forms) two way data binding might be preferred.
 
Comparing to pure react: (with state managed through useState/useContext) - better separation of concerns. VMs could be threaten like the container (stateful) components, hovewer they do not render anything and subsequently it is much harder to mess up the state management logic and UI code. Avoid the neccessity to care about hooks sequence, about keeping the useEffects dependencies tracked in the 'deps' array, about keeping an information about whether the component is still mounted when asyn action is completed, about making sure closures from 'previous' renders are not suddenly used during effect execution. 
As any other tech, hooks (and in particular - useEffect) requires developer to follow some recommendations (not declared in the interface, but accepted as a 'approach', 'mental model' or 'best practices'). There is a nice guide to useEffect https://overreacted.io/a-complete-guide-to-useeffect/ from one of react team members. Read it and then answer two questions: 
1) what do you acheive with hooks? 
2) how many requirements, not controlled by compiler and hardly tracked by linter/visual code review, should be met by developer while working with hooks in order to make them predictable.
Once the second list exceedes the first one by a huge margin - it is a good sign to me that an approach should be considered with a huge caution.
hint: tiny bit of frustratino about hooks https://blog.logrocket.com/frustrations-with-react-hooks/


Comparing to react-mobx integration described in https://mobx.js.org/react-integration.html
Code structuring is not defined within the shipped in react-mobx docs. It is up to the development team. You may consider an approach, described in this article as an approach to structure the application written with react for rendering and mobx for state management.

Comparing to mobx-state-tree: Viewmodels are native class instances without framework-specific syntax blended in. Mobx-state-tree types definitions heavily rely on the framework specific syntax. Also mobx-state-tree forces the duplication of code while declaring the types because properties of particular type should be described both in the typescript interface and in the model definition. 