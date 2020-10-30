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

* Keep the keys always in uppercase
  Bad
  ```json
    {
      "home": "home"
    }
  ```
  Good
  ```json
    {
      "HOME": "home"
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
 
  
  
  
  
  
  
  
  
  
  
