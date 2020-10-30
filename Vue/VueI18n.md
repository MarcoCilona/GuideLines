# VueI18n

VueI18n is a third party library that is used to provides translations file management into your Vue application.

### Basic Configuration

#### i18n.js
```js
import Vue from 'vue';
import VueI18n, { LocaleMessages } from 'vue-i18n';

Vue.use(VueI18n);

/**
 * Scans the src/locales directory for JSON files, and loads them in as translations messages.
 */
function loadLocaleMessages(): LocaleMessages {
  const locales = require.context('./locales', true, /[A-Za-z0-9-_,\s]+\.json$/i);
  const messages: LocaleMessages = {};
  locales.keys().forEach((key) => {
    const matched = key.match(/([A-Za-z0-9-_]+)\./i);
    if (matched && matched.length > 1) {
      const locale = matched[1];
      messages[locale] = locales(key);
    }
  });
  return messages;
}

/**
 * Creating VueI18n instance with its configurations
 */
export default new VueI18n({
  fallbackLocale: process.env.VUE_APP_I18N_FALLBACK_LOCALE || 'en',
  locale: process.env.VUE_APP_I18N_LOCALE || 'en',
  messages: loadLocaleMessages()
});
```

As you can see, the fallbacklocale and locale params are retrieved from the .env file. This will mantain the configurations of the vue app stored in one place and easy to retrieved.

#### Usages
  * $t: when you need to use a translation from a file, use $t function. It takes the key representing the translation as argument. If the label is nested, the key must be the complete 'path' to that translation.
    ```json
    {
      "GREETINGS": {
        "HELLO": "hello"
      },
      "HOME": "home"
    }
    ```
    ```js
      this.$t('HOME')
      this.$t('GREETINGS.HELLO')
    ```
  * $tc: when pluralization is needed use $tc to translate that label. It takes the key representing the translation and the number of occurrencies as argument. 
    ```json
    {
      "CAR": "car | cars"
    }
    ```
    ```js
      this.$tc('CAR', 1) ----> 'car'
      this.$tc('CAR', 2) ----> 'cars'
    ```
  * text interpolation: sometimes you will need to pass arguments to the translation. Use the text interpolation to do so
    ```json
    {
      "NAME": "my name is {name}"
    }
    ```
    ```js
      this.$t('NAME', { name: 'James' }) ---> 'my name is James'
    ```
#### Translations files

The translations files should be placed in './src/locales' and there must be a file for every language the application is going to support.
VueI18n supports different file format, but the '.json' files are preferred because they are easy to read.
So, if for example we are storing translations for both 'en' and 'de' language, the folder structure will be something like this:

```bash
├── src
│   ├── locales
│   │   ├── en.json
│   │   ├── de.json
```

##### File structure

Follow this guide lines when creating a translation file:

* Split the translations into categories. One of the main problem using VueI18n is the duplication of labels. One approach to avoid or minimize this problem is to split the translations into categories.
  - GLOBALS: all the labels that are use across the entire application (like: 'Next', 'Home', 'Login', 'Go back', and so on). Every time a label is used in multiple places in the application and it's without a context, put it under GLOBALS
    ```json
    {
      "GLOBALS": {
        "GO_BACK": "go back",
        "NEXT": "next"
      }
    }
    ```
  - INFO: all the labels that identify a generic info should be place here. Imagine you make an api call to BE and based on the response you show a generic info message like: 'User updated', or 'Failed to save the user'. Those messages can also be shown when saving a post, ora saving an image. Those generic, dynamic translations should be put under INFO category.
    ```json
    {
      "INFO": {
        "DELETE": "Are you sure you want to delete {item}?",
        "ERROR": "a problem occurred while saving {item}",
        "SUCCESS": "{item} saved correctly!"
      }
    }
    ```
  - UI: all the labels related to ui elements should be placed here. For example, all the labels used in a form submition like 'submit', 'reset', 'username', ecc...
  - Create a category for every available application macro domain. If your application has, for example, a dashboard, a section to handle the user information and a section to handle its post, three categories can be defined 'DASHBOARD', 'USER' and 'POST'.
In case that one of the macro domain has subsections, think about nesting it under the main category.
Here an example:
    ```json
    {
      "DASHBOARD": {
        "WELCOME": "welcome to awsome.it"
      },
      "POST": {
        "DESCRIPTION": "this post is about {topic}",
        "ACTIONS": {
          "SHARE": "share {postTitle} on facebook to let your friends know what you've just learned!"
        }
      }
    }
    ```
* Keep the keys always in uppercase and use underscores to separate multiple words when needed
  Bad
  ```json
    {
      "home": "home",
      "my-house": "my house
    }
  ```
  Good
  ```json
    {
      "HOME": "home",
      "MY_HOUSE": "my house
    }
  ```
 * Keep the labels always lower case: when creating a translation you must focus only on it and not on its format. So keep every label lowercase, if some formatting  is needed, it will be applied with css or javacript.
   Bad
   ```json
     {
       "HOME": "Home"
     }
   ```
   Good
   ```json
     {
       "HOME": "home"
     }
   ```
 * Use placeholders: sometimes you will need a string with dynamic content, don't concatenate different string to achieve the final one, but use placeholders instead because, sometimes, the same sentence may have a different word positioning in different languages. And, using placeholders, you can also have dynamic text inside the phrase and not only on its end or start.

   ```json
     {
       "FAVORITE_COLOR": "my name is"
     }
   ```
   ```js
    const finalString = this.$t('FAVORITE_COLOR') + 'Ermenegildo'
   ```
   Good
   ```json
     {
       "FAVORITE_COLOR": "my name is {color}"
     }
   ```
   ```js
    const finalString = this.$t('FAVORITE_COLOR', { color: this.$t('RED')})
   ```
  * Keep alphabetical order: when creating a new translation, keep the alphabetical order
    Bad
    ```json
      {
        "HOME": "home",
        "CALL": "call"
      }
    ```
    Good
    ```json
      {
        "CALL": "call",     
        "HOME": "home"
      }
    ```
   * Labels key in english: the key of every label must be in english
     Bad
     ```json
       {
         "CASA": "home"
       }
     ```
     Good
     ```json
       {    
         "HOME": "home"
       }
     ```
  * If you are creating a translation for a single word use pluralization when possible this will avoid label duplications if plural or single version is needed later on
    Bad
    ```json
      {
        "CAR": "car"
      }
    ```
    Good
    ```json
      {    
        "CAR": "car | cars"
      }
    ```
  * Use linked translations: sometimes you will need some translation to use another translation into it. Use the linked translations instead.
    Bad
    ```json
      {
        "NOT_FOUND": "not found",
        "PAGE": "page",
        "PAGE_NOT_FOUND": "page not found"
      }
    ```
    Good
    ```json
      {
        "NOT_FOUND": "not found",
        "PAGE": "page",
        "PAGE_NOT_FOUND": "@:PAGE @:NOT_FOUND"
      }
    ```
    The linked translations also supports string formatting, so, if you need one of the linked translation to be, for example, uppercase the linked string will be like this: "PAGE_NOT_FOUND": "@.uppercase:PAGE @:NOT_FOUND".
The available modifiers are:
      - uppercase
      - lowercase
      - capitalize
    
    **N.B.: use this kind of linked translation only if the linked one is static and won't be change dynamically. If, for example, you have a translation 'My favorite color is' and then a dynamic value for the color is needed, use the text interpolation**

  

  

