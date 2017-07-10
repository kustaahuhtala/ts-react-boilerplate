# Completing React

Now we finally get to build our **React**-application into something that can be run and will actually do something! Here we make the assumption that you are going to build a medium- to large-sized application and show how to do these things in a more modular way, but for smaller applications some of these parts can be merged with the `IndexView` and some can be left out completely.

### Initialize

We begin by installing new dependencies called [React Router](https://reacttraining.com/react-router/) and [ReactDOM](https://facebook.github.io/react/docs/react-dom.html)
```
yarn add react-router-dom react-dom
```
which adds the capability to define which URL equals which view and to render our **React** application to the [DOM](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model/Introduction), respectively.

And of course the types for them
```
yarn add -D @types/react-router-dom @types/react-dom
```

### AppView

We begin by writing an `AppView.ts` file into the `src/modules`-folder
```typescript
import * as React from 'react';
import { Route, Switch, RouteComponentProps } from 'react-router-dom';
import IndexContainer from './index/IndexContainer';
import PageNotFound from '../components/PageNotFound';

export type IAppViewProps = RouteComponentProps<undefined>;

const AppView: React.StatelessComponent<IAppViewProps> = () => (
    <div className="app-base">
        <Switch>
            <Route path="/" exact component={IndexContainer} />
            <Route component={PageNotFound} />
        </Switch>
    </div>
);

export default AppView;
```
which is a fairly simple view, except for the `<Switch>`-clause and `RouteComponentProps`.
> All views that go through **React Router** get some extra properties, which are included in `RouteComponentProps<Params`, where `Params` can be used to define possible parameters for the URL.

The [`<Switch>`](https://reacttraining.com/react-router/web/api/Switch) element is used to render a single view out of all [`Route`s](https://reacttraining.com/react-router/web/api/Route) inside the `<Switch>`. `Route`s have three main properties you should remember:
1. `path` which indicates what URL the view matches to (*it can be used to define parameters as well*)
2. `exact` which indicates that the view should only match if the URL matches `path` exactly
3. `component` which defines the actual view to render

### AppContainer

Next we create a very simple **container** for `AppView` called `AppContainer`, which is situated in the same `src/modules`-folder
```typescript
import { connect } from 'react-redux';
import AppView, { IAppViewProps } from './AppView'; //tslint:disable-line:no-unused-variable

export default connect<{}, undefined, IAppViewProps>(() => ({}))(AppView);
```
which we use just to wrap `AppView` so that it can be used in routes.

### IndexView

For `IndexView` we also need to add `RouteComponentProps`, so just add the following
```typescript
import { RouteComponentProps } from 'react-router-dom';
// ...
export type IIndexProps = IIndexState & IIndexDispatch & RouteComponentProps<undefined>;
```

### index

Finally we create a file `index.ts` inside `src`
```typescript
import * as React from 'react';
import * as ReactDOM from 'react-dom';
import { Provider } from 'react-redux';
import { Route } from 'react-router-dom';
import { ConnectedRouter } from 'react-router-redux';
import createHistory from 'history/createBrowserHistory';
import configureStore from './redux/store';
import AppContainer from './modules/AppContainer';

const history = createHistory();

ReactDOM.render((
    <Provider store={configureStore(history)}>
        <ConnectedRouter history={history}>
            <Route component={AppContainer} />
        </ConnectedRouter>
    </Provider>
    ), document.getElementById('app'),
);
```
which is the entry file to our application that ties everything together.

---

On the 12. line
```typescript
import * as ReactDOM from 'react-dom';

ReactDOM.render((
    // ...
    ), document.getElementById('app'),
))
```
we [render](https://facebook.github.io/react/docs/react-dom.html#render) our **React**-application to the **DOM** inside a div with the id `app` (*we'll come back to this*).

---

On the 13. line
```typescript
import * as React from 'react';
import { Provider } from 'react-redux';
import createHistory from 'history/createBrowserHistory';
import configureStore from './redux/store';
const history = createHistory();
// ...
    <Provider store={configureStore(history)}>
        // ...
    </Provider>
```
which wraps our **React** application with **Redux** using [`Provider`](https://github.com/reactjs/react-redux/blob/master/docs/api.md#provider-store) from **react-redux**, which takes a single parameter `store`, for which we provide our store as we defined it in [Redux](/REDUX.md#store). `history/createBrowserHistory` is used to create a wrapper around the browser history we can use.

---

On the 14. line
```typescript
import * as React from 'react';
import { Route } from 'react-router-dom';
import { ConnectedRouter } from 'react-router-redux';
import AppContainer from './modules/AppContainer';
// ...
        <ConnectedRouter history={history}>
            <Route component={AppContainer} />
        </ConnectedRouter>
// ...
```
we keep the UI in sync with the URL using a [`ConnectedRouter`](https://github.com/ReactTraining/react-router/tree/master/packages/react-router-redux) from **react-router-redux**, which takes as argument a [`history`](https://github.com/ReactTraining/react-router/blob/v3/docs/API.md#histories), where we give `history` we created previously. Here we define a single `Route` which renders `AppContainer` for all URL routes.

### Index.html

Finally we write an `index.html` in our root-folder
```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8" />
        <title>Todo app</title>
    </head>
    <body>
        <div id="app"></div>
        <script src="js/bundle.js"></script>
    </body>
</html>
```
which is just a very simple `HTML`-file, which imports our (*soon-to-be-bundled*) **JavaScript** from the path `/js/bundle.js` and contains a `div` with the id `app` so our `index.ts` works.

### Scripts

Now as we have everything necessary for our application, it's time to get it working!

We'll begin by installing a couple of new dependencies
```
yarn add -D browserify budo tsify
```
from which [browserify](http://browserify.org/) allows us to do `import`-statements in client code, [budō](https://github.com/mattdesl/budo) is a lightweight server to host client code (*it uses browserify under the hood*) and [tsify](https://www.npmjs.com/package/tsify) is a **browserify**-plugin to use it with **TypeScript**.

---

First it's time to write our development script, so head on over to your `package.json` and add the following
```json
    // ...
    "scripts": {
        "develop": "budo src/index.tsx:js/bundle.js --live --verbose -- -p tsify"
    }
```
which allows you to start the **budō** server by entering `yarn run develop` into your console (*inside the root folder of your app*). It will run until it faces an error it can't recover from or you press `ctrl+c`. The arguments given to **budō** are firstly, the name of your application entry file (*with a relative path*), followed by a double colon with the path and name of your `index.html` (*relative to root folder*) expects to find your application code. `--live` enables [LiveReload](http://livereload.com/) (*you can install a plugin to [Chrome](https://chrome.google.com/webstore/detail/livereload/jnihajbhpnppcggbcgedagnkighmdlei?hl=en) or [Firefox](https://addons.mozilla.org/en-gb/firefox/addon/livereload/) to take full advantage of it*), which automatically refreshes your browser when the source code changes. `--verbose` enables verbose output from **budō** and then all arguments after the ` -- ` will go to the **browserify** **budō** is running, in this case a plugin **tsify** to compile our **TypeScript** files.

---

Now it's time to build our application to be hosted on the Internet, so add the following line to your `scripts` inside `package.json`
```json
    // ...
    "scripts": {
        // ...
        "build": "mkdir -p dist/js && browserify src/index.tsx -p tsify > dist/js/bundle.js"
    }
```
which will first make sure that the folder `dist/js` is there and then build your application with **browserify**. For **browserify** we give as first argument the entry file, then a plugin (*again, **tsify** to use **TypeScript** with **browserify***) and finally after the `>` the output file name (*with its relative path*). Apart from this script (*run by `yarn run build`*) you only need to copy your `index.html` file to the `dist`-folder and your application is complete!

### Alternatives

- An alternative for **browserify** is [webpack](https://webpack.github.io/), which is maybe a bit more popular these days, but I personally dislike the amount of configuration (*and the way the configuration is achieved*) it requires