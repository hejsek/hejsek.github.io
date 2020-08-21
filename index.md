## How to display Angular 2 applications in the user’s browser language

Our mission today is to dynamically translate your Angular 2+ application to the user’s browser language. I tried to develop this solution with an emphasis on performance and simplicity. 

Language translation format:
We are going to store our language versions in JSON format. I decided to use this because you can easily convert JSON to objects and afterwards work with it easily.

```javascript
{
   "en": {
       "work": "Work",
       "hello": "Hello"
   },
   "de": {
       "work": "Arbeit",
       "hello": "Allo"
   }
}
```

I think that this is kind of self-explanatory.

Ok, after we have prepared our language versions for English and German we will actually be coding something! Angular has a cool feature called Pipes. Pipes basically transforms input data into a desired output. In our case the pipe will convert the input string into the translation of that string. 
Afterwards you can simply translate the word “hello” in templates like this:

```html
<h1>
 Translation: {{"hello" | translate}}
</h1>
```
That’s all. Sounds simple, right? Well it is.

First, we must create a file called **translations.pipe.ts**. If you are using angular-cli, which I strongly suggest you all do, you will have to type “ng generate pipe translations” into the command line in your angular-cli project folder.

After successfully creating the desired pipe we can start to code. This is the head of our code: 
```typescript
// Import Angular Pipe Utilities
import {Pipe, PipeTransform} from "@angular/core";

// Enter the keyword to invoke the Pipe in templates via @Pipe decorator
@Pipe({
 name: 'translate'
})

export class TranslationsPipe implements PipeTransform {
  /*
    Our code will be here
  */
}
```

Now for the actual functionality.
Every Pipe must have a method called *transform*, with the following format: **transform(value: any, args?: any)**. When the pipe is applied to data, the data are always sent to the *transform()* method where they are processed and returned. 

This was a simple introduction to how Pipes work and now we can get to the code itself.
I tried to add comments to every piece of code so it should be easily understandable for everyone.

Our **translations.pipe.ts** will look like this:

```typescript
// Import Angular Pipe Utilities
import {Pipe, PipeTransform} from "@angular/core";

// Create name of our pipe used in templates via @Pipe decorator
@Pipe({
 name: 'translate'
})

// Pipes itself
export class TranslationsPipe implements PipeTransform {
 // Default Pipe input method
 transform(inputString: string): string {
   // Get current browser language in the form of a short string. For example 'en', 'fr' etc.
   const language = 'en';

   // Get translations for given language
   const translationsForLanguage = this.translations(language);

   try {
     // Parse string formatted in JSON to object for given language
     const translation = translationsForLanguage[inputString];

     // If translation does not exist in this language, the original input string is returned
     if (typeof translation === 'undefined') {
       console.log("Translation wasn't found for string " + inputString);
       return inputString;
     }

     return translation;
   } catch (e) {
     console.log("Translation wasn't found for string " + inputString);
     return inputString;
   }
 }

 // returns JSON object in given language
 translations(language: string) {
   // Translations in JSON format
   const json = TranslationsPipe.translationsJson();

   // Parse JSON string to object
   const translations = JSON.parse(json);

   // Get translations only for specified language
   const translationsForLanguage = translations[language];

   // If translations for input language wasn't found, the English version is returned
   if (typeof translationsForLanguage === 'undefined') {
     console.log("Translations for language " + language + " wasn't found.");
     return JSON.parse(json)['en'];
   }

   return translationsForLanguage;
 }

 // This is a static function where our translations are stored
 static translationsJson() {
   const json = `
       {
           "en": {
               "work": "Work",
               "hello": "Hello"
           },
           "de": {
               "work": "Arbeit",
               "hello": "Allo"
           }
       }
       `;

   return json;
 }
}
```

******Note*****
If you are not using angular-cli you must set a declaration in your **app.module.ts** under @NgModule directive.

It will look like this:
```typescript
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { FormsModule } from '@angular/forms';
import { HttpModule } from '@angular/http';

import { AppComponent } from './app.component';
import { TranslationsPipe } from './translations.pipe';       //<---------- This must be added manually

@NgModule({
 declarations: [
   AppComponent,
   TranslationsPipe       //<---------- This must be added manually
 ],
 imports: [
   BrowserModule,
   FormsModule,
   HttpModule
 ],
 providers: [],
 bootstrap: [AppComponent]
})
export class AppModule {
}
```

Hope it helps!

