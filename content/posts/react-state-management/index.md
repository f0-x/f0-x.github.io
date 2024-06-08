+++
title = "How I Approach State Management on React"
publishDate = 2024-06-08T00:00:00+01:15
lastmod = 2024-06-10T11:24:47+05:45
tags = ["reactjs", "javascript"]
categories = ["frontend", "programming"]
draft = false
featuredImage = "images/react-state-options.jpg"
featuredImagePreview = "./posts/react-state-management/images/react-state-featured-image.avif"
+++

Us React developers, we often find ourselves grappling with the age-old question:
{{< admonition question "Which state management approach should you use ?" false >}}
It depends 🤡
{{< /admonition >}}
React offers a variety of state management solutions, each with its own strengths and weaknesses, and the choice ultimately depends on the specific requirements of your project.
To help navigate this decision, I follow this self-made flow chart which serves as an excellent guide, taking into account various factors such as data sharing requirements, data fetching needs, and whether you're dealing with forms or multiple components.

{{<mermaid>}}
flowchart TD
    start[Picking a React State Approach] -->|No| isForm{Form?}
    isForm -->|Yes| formState["fa:fa-address-book Form state"<br>useState<br>useReducer<br>Formik<br>React Hook Form<br>Tanstack Form]
    isForm -->|No| isMultipleComponents{For multiple components?}
    isMultipleComponents -->|No| plainState["fa:fa-image Plain state"<br>useState<br>useReducer]
    isMultipleComponents -->|Yes| isFetched{Fetched?}
    isFetched -->|No| propDrillOrGlobalState["fa:fa-globe Prop drill or global state"<br>localStorage<br>cookie<br>IndexedDB<br>Redux<br>Zustand<br>Recoil<br>Jotai<br>Valtio]
    isFetched -->|Yes| isShareableViaURL{Sharable via URL?}
    isShareableViaURL -->|No| isMultipleComponents2{For multiple components?}
    isMultipleComponents2 -->|No| routeState["fa:fa-bookmark Route state"<br>React Router loader<br>Tanstack Router loader]
    isMultipleComponents2 -->|Yes| remoteState["fa:fa-at Remote state"<br>Tanstack Query<br>swr<br>RTK Query<br>Apollo]
    isShareableViaURL -->|Yes| useURL["fa:fa-bookmark Use URL"]

    classDef startNode fill:#FAB005,stroke:#734200,stroke-width:2px,color:#000;
    classDef decisionNode fill:#FFE0B2,stroke:#734200,stroke-width:2px,color:#000;
    classDef solutionNode fill:#5FC78B,stroke:#1D5C36,stroke-width:2px,color:#000;

    start:::startNode;
    isForm:::decisionNode;
    isMultipleComponents:::decisionNode;
    isFetched:::decisionNode;
    isShareableViaURL:::decisionNode;
    isMultipleComponents2:::decisionNode;
    formState:::solutionNode;
    plainState:::solutionNode;
    propDrillOrGlobalState:::solutionNode;
    routeState:::solutionNode;
    remoteState:::solutionNode;
    useURL:::solutionNode;

    linkStyle 0 stroke:#734200,stroke-width:2px;
{{</mermaid>}}

### Plain State 📃

If your application has simple state management needs and doesn't require data sharing across multiple components, the built-in [`useState`](https://reactjs.org/docs/hooks-state.html) or [`useReducer`](https://reactjs.org/docs/hooks-reference.html#usereducer) hooks, along with component state (`useState` used within a functional component), can suffice. These hooks provide a straightforward way to manage component-level state without the overhead of additional libraries or abstractions.

### Form State 📝

When dealing with forms, dedicated form state management libraries like `useReducer`, [`Formik`](https://formik.org/), [`React Hook Form`](https://react-hook-form.com/), [`Tanstack Form`](https://react-hook-form.com/), or [`XState`](https://xstate.js.org/) can be extremely helpful. These libraries offer robust form handling capabilities, including validation, error handling, and performance optimizations specific to form data management.

### Shared State 🌐

If your application requires sharing state across multiple components, you have several options:

#### Prop Drilling or Global State 🌎

For simple shared state scenarios, you can leverage prop drilling (passing data down the component tree via props) or global state management solutions like `localStorage`, `cookie`, `IndexedDB`, [`Redux`](https://redux.js.org/), [`Zustand`](https://github.com/pmndrs/zustand), [`Recoil`](https://recoiljs.org/), [`Jotai`](https://github.com/pmndrs/jotai), [`Valtio`](https://github.com/pmndrs/valtio), or [`XState`](https://xstate.js.org/). These solutions allow you to store and access shared state across your application.

#### Route State 📚

If the shared state is specific to a particular route or set of routes, you can leverage the built-in state management capabilities of React Router, such as the `React Router loader` or `Tanstack Router loader`. These solutions are particularly useful when working with data that needs to be fetched and shared across multiple components within a specific route.

#### Remote State 📧

For applications that require fetching and sharing data from remote sources, you can use libraries like [`Tanstack Query`](https://tanstack.com/query/latest), [`SWR`](https://swr.vercel.app/), [`RTK Query`](https://redux-toolkit.js.org/rtk-query/overview), or [`Apollo`](https://www.apollographql.com/) (for GraphQL). These libraries provide efficient data fetching, caching, and state management for remote data, reducing the need for manual cache invalidation and improving overall performance.

### Sharable via URL? 🔗

One important consideration when choosing a state management approach is whether the state needs to be shareable via URL. If so, solutions like `URL` state management or the built-in capabilities of React Router become more relevant.

By following this flowchart, you can systematically evaluate your application's state management needs and choose the most appropriate solution. Remember, there's no one-size-fits-all approach, and you may need to combine multiple strategies to meet your application's requirements effectively.

{{< admonition type=tip title="You should remember this!" open=true >}}
State management is a crucial aspect of building complex React applications, and making an informed decision can significantly impact the maintainability, scalability, and performance of your codebase. By leveraging the guidance provided in this flowchart, you can navigate the state management landscape with confidence and build robust, efficient React applications.
{{< /admonition >}}
