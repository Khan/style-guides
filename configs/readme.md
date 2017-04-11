# style and linting configuration

## android studio

komalo or nacho know what to do with the `KhanAcademyAndroid.xml` file.

## javascript/react

you can get eslint reminding you to write better code as you edit by setting up
this linter.

It's pretty straight forward! `npm install` this and use this `.eslintrc` and
you'll get a fancy pants eslint setup that you can integrate with your favorite
editor!

but if you don't like this package.json nonsense, you can do

    npm install --save-dev eslint
    npm install --save-dev babel-eslint
    npm install --save-dev eslint-plugin-react

and that will basically do the same thing. you're in the clear as long as the
bundled `.eslintrc` is in a root dir of your webapp.

