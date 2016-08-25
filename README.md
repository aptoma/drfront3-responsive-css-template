# drfront3-responsive-css-template
General styling to make fronts in DrFront 3 into responsive fronts

### Dependencies for development

- ruby 2.1.2 ([For installation instructions see: [rvm](https://rvm.io/rvm/install))
- bundler (For installation instructions see: [Getting Started](http://bundler.io/))

```sh
 rvm install 2.1.2
 rvm use 2.1.2
 bundle install
 compass compile
```

Images in DrFront 3 is not responsive! You have to change the image dimentions and remove the width and height attributes on the images meant for mobile.

To do this, you can use our image transformation services to transform your desktop images into mobile. See: [Aptoma Smooth Storage documentation](documentation/ass.md)

If you are interested in node development we also have a [Aptoma Smooth Storage client](https://github.com/aptoma/ass-client-js)

NB: Please contact us in good time before deploying your solution into production to avoid any surprises.
