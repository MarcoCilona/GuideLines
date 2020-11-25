## VUE

#### Best Practices
* **PascalCase** Use the PascalCase for the vue components name and files
* **Component naming** If the component is used once and JUST once in a view naming the component with the prefix The (TheModalConfirmAction)
    If the component is a basic component such as a button, a select list, prefix the name with the word Base. (BaseButton, 
    BaseTable, BaseSelect...).
* **Events naming** when creating a custom event, put the name in kebab-case.
    The event name should be composed as follows:
    __object-name__-__action-made__.
    Ex.: button-clicked.
    The event callback name should be construct as follows:
    handle __eventNamePascalCase__
    ex.: handleButtonClicked
    If more than one event with the same name is used inside the page, add some attribute specific to the element to the event name.
    ex.: handleConfirmButtonClicked
* **Use mixins** if some components shares the same code, export it in a mixin. This way you will be able to reuse that code as the component property and use it anywhere.
* **Component style** prefer SFC. If you have to move the style out of the component file, named that file with the component name. Ex.: Button.vue - Button.scss
* **Props creation** when creating a component prop use camelCase and follow this pattern: propName: { default: default-value, type: propType }
* **Create methods call** if a method is called both in created hook and in a watched element, remove the call from the created hook, its useles
* **Loop key** always add key attribute when using loop inside the template
* **Object.freeze** if an array is used to show information in the template and it'll no be changed, use Object.freeze to prevent you to make it reactive
* **$set** use $set when you need to update an object nested property. ex.: this.$set(object, propName, newValue)
* **$ref** don't overuse $ref to call child elements methods. Use instead event-props pattern
* **Nested child events** if you need to share information across deeply nested components, think about using an event bus or, better, Vuex store
* **Small components** keep components as small as possible. Don't put too much logic inside the same components. This will prevent from releasing bug and make refactor easier
* **Don't use v-if along with v-for**
* **Don't mix component logic with view** if something can be achieved with css rules use them. Try not to query the dom to add styles or classes
* **Use $refs instead of querySelectors** when you need to make some operation on an element or call a component method, give it a ref and use the 'this.$refs' property of the Vue instance
* **Use slot** if a component starts having too much logic to handle its template, think about using slot or scoped-slot to display custom data from outside the component, leaving it as simple as possible and resusable


##### Vuex
When using Vuex we are storing information inside its state.
To update this state we will use two of the main Vuex core concept:
* action: will contain some logic and then call a mutation to update the state;
* mutation: will update the state

If we want to update the state from one component, call the dispatch function passing two arguments, the action to be dispatched and the value we want to be stored.

The action will make some operations, if needed, and than call a mutation to upadate the value.

To access a store state variable set a getter, this way in every component that variable will be accessible.

If the store grows in size, it's better to split it into modules, each of one is specific to some domain.
By importing this module inside a index.js file, the store will be more readable and maintanable.

The project will have a folder called store with the following structure:

store/
|
|- index.js
|- modules/
    |- module1.store.js 
    |- module2.store.js 

Here an example of how to structure a store module:

```js
export const state = {
  items: []
};

/**
 * Mutates the state
 */
export const mutations = {
  addItem(state, item) {
    state.items.push(item);
  },
  removeItem(state, index) {
    state.items.splice(index, 1);
  }
};

/**
 * Made some operations and then call a mutation
 */
export const actions = {
  updateArray({ commit, state }, item) {
    if (state.items && state.items.indexOf(item) !== -1) {
      commit("removeItem", state.items.indexOf(item));
    } else {
      commit("addItem", item);
    }
  }
};

const getters = {
  items: state => state.items
};

export default {
  state,
  actions,
  mutations,
  getters
};
```

##### Vue router
Create a route for every page of the application. The routes will be an array of route object, structured as follows:

{
    alias: "",
    path: "",
    name: "",
    component: () => import(""),
    children: [],
    redirect: ''
}

Use __alias__ to provide an alternative alias to a single route.
Use __path__ to define the url to which the component will be mapped.
Use __name__ to reference the route without having to put the entire path.
Use __component__ to link the component to the url. Use the dynamic import as shown in the example to keep the performance high.
If the application has some sections with common components, for example a navbar, use vue router __children__ to have a main component with the navbar and some sub-route for the child components. In the __children__ the child components will be mapped.
Use __redirect__ if, once the route has a default sub-route to be redirect to.

###### User role and authentication
Use the guard 'beforeEnter' to made some check on the user and prevent accessing that route if not authorized.

##### Modules
To prevent the route file to become too much large and creating confusion, we can split the routes into sub files based on their domains. This way we will have a single entry point, index.js, in which the roues will be imported.

Example of structure:

router/
|
|- index.js
|- modules/
    |- administration.js
    |- dashboard.js


##### Axios
Use VueAxios to handle api calls.
The best way to manage axios is to create an __api.service.js__ where the axios instance is created and initialized as an object with all the main HTTP methods.

The axios instance will look like this:
```js
const ApiService = {
  init() {
    Vue.use(VueAxios, axios);
    Vue.axios.defaults.baseURL = API_URL;
  },

  get(resource, slug = "") {
    return Vue.axios.get(`${resource}/${slug}`).catch(error => {
      throw new Error(`[RWV] ApiService ${error}`);
    });
  },

  post(resource, params) {
    return Vue.axios.post(`${resource}`, params);
  },

  put(resource, params) {
    return Vue.axios.put(`${resource}`, params);
  },

  delete(resource) {
    return Vue.axios.delete(resource).catch(error => {
      throw new Error(`[RWV] ApiService ${error}`);
    });
  }
};
```

As the application could be very big, having lots of api call, we can use __api.service.js__ as an entry point in which initialize the VueAxios instance with its main methods.
The other api call will be stored in some specific files, based on their context.
For example, all the api with enpoint 'user', will be stored in a user.service.js file. This file will be later imported in the __api.service.js__ and made available in the application with an export statement.

Ex.:
```js
const ApiService = {
  init() {
    Vue.use(VueAxios, axios);
    Vue.axios.defaults.baseURL = API_URL;
  },

  get(resource, slug = "") {
    return Vue.axios.get(`${resource}/${slug}`).catch(error => {
      throw new Error(`[RWV] ApiService ${error}`);
    });
  },

  post(resource, params) {
    return Vue.axios.post(`${resource}`, params);
  },

  put(resource, params) {
    return Vue.axios.put(`${resource}`, params);
  },

  delete(resource) {
    return Vue.axios.delete(resource).catch(error => {
      throw new Error(`[RWV] ApiService ${error}`);
    });
  }
};

export ApiService; // This is needed to be able to access ApiService instance from the other submodules of the api.service
export { UserService } from "./api-resources/user.service"; // 
```

A generic service file will be structured like this:
```js
import ApiService from "@/common/api.service";

const RES_USER = "user";

export const UserService = {
  getUser(payload) {
    return ApiService.get(`${RES_USER}`);
  }
};
```

###### Folder structure
common/
|
|- config.js
|- api.service.js
|- api-resources/
    |- user.service.js
    |- project.service.js
    |- ...

###### Best Practices
* **File name** every service file will be called with the following pattern: __domain__ service.js.
* **Endpoint storage** in every service file save the used api endpoints in some const variable.
Bad
```js
import ApiService from "@/common/api.service";

export const UserApi = {
  getUser(payload) {
    return ApiService.get('user');
  }
};
```

Good
```js
import ApiService from "@/common/api.service";

const RES_USER = "user";

export const UserService = {
  getUser(payload) {
    return ApiService.get(RES_USER);
  }
};
```
* **Create descriptive name** when creating a method making an api call, use a descriptive name to tell what kind of operation will be done. ex.: getUser, saveUser, updateUser, ecc...

##### Project structure
A Vue project src folder should be structured as follows

src/
|
|- assets/
|- common/
    |- api.service.js
    |- utils.js
    |- api-resources/
        |- user.service.js
        |- ...
|- components/
    |- base/
    index.js
|- locale/
|- mixins/
|- router/
    |- index.js
    |- admin.js
    |- ...
|- store/
    |- index.js
    |- modules/
        |- user.module.js
|- styles/
|- views/
|- App.vue
|- main.js

Description
* **assets**: store images, svg and media files;
* **common**: store api service (VueAxios), utils functions (helper functions);
* **components**: stores the components used inside the application. The base components should be separated from the others, saved in a subfodler called 'base'. If the components grows in numbers, think about split them by their domain and store them in subfolders. The __index.js__ file is used to export all the components created;
* **locale** stores the VueI18n translations;
* **mixins** if the applications make use of some mixins, store them inside this folder;
* **router** stores the applications routing system. The index.js file will be the entry point, importing all the routes modules;
* **store** put all the information about the Vuex store inside this folder;
* **styles** stores the application styles (css or scss);
* **views** when a component is used to make a page layout, its considered a view component. Commonly those components are the one used by VueRouter. When this folder grows in size, think about splitting the views into subfolders based on their context. ex.: all the views related to the admin page will be stored inside a __admin__ folder;
