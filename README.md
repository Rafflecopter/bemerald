<hr>

# NOT MAINTAINED

<hr>

<img src="http://f.cl.ly/items/2Q1I3z1x3C3W1J2t0D1c/bemerald.svg" align=right height=200>

# BEMerald

Pure-sass functions and mixins to make writing [BEM](http://getbem.com/naming/) a piece of cake. Keeps things simple and handles tricky cases gracefully.
<br>

- **✓** **[lib]Sass ≥ 3.4**<br>
- **✗** &nbsp;~~[lib]Sass < 3.4~~

## tl;dr

```scss
/** simple **/

@include Block(notification) {     // .notification {
  border: 1px solid #ccc;          //   border: 1px solid #ccc;
  padding: 20px;                   //   padding: 20px;
                                   // }
  @include Elem(link) {            // .notification__link {
    float: right;                  //   float: right;
  }                                // }
                                   // .notification--error {
  @include Mod(error) {            //   border-color: red;
    border-color: red;             // }
  }
}                                  
                              

/** more involved **/ 

@include B(myblock) {              //  .myblock {
  background: white;               //    background: white;
                                   //  }
  @include E(subelem) {            //  .myblock__subelem {
    color: blue;                   //    color: blue;
  }                                //  }
                                   //  .myblock:hover {
  &:hover {                        //    text-decoration: underline;
    text-decoration: underline;    //  }
                                   //  .myblock:hover .myblock__subelem {
    @include E(subelem) {          //    color: green;
      color: green;                //  }
    }                              //  .myblock--mymod {
  }                                //    background: green;
                                   //  }
  @include M(mymod) {              //  .myblock--mymod .myblock__subelem {
    background: green;             //    color: green;
                                   //  }
    @include E(subelem) {          //  .myblock--mymod:hover {
      color: green;                //    font-weight: bold;
    }                              //  }
                                   //  .myblock--mymod:hover .myblock__subelem {
    &:hover {                      //    color: red;
      font-weight: bold;           //  }
      
      @include E(subelem) {
        color: red;
      }
    }
  }
}
```

## Installation

Right now you have 2 options:
- Copy/paste the `bemerald.scss` file (it's all one self-contained file) into your project
- Install via `npm install bemerald` and configure your builder to point to the `node_modules` folder

**TODO**: Provide easier installation & plugins for common build tools

## Usage

_in order of most commonly used..._

### Mixins

The build-in mixins allow you to write your SCSS in a nested way that feels very natural, but still get flat root-level selectors, as per BEM methodology.

-----

#### `@include Block($block) { ... }`

Simply unrolls into a class-selector. Main purpose of using this mixin is to clearly denote the start of a BEM block.

```scss
@include Block(notification) {     //  .notification {
  position: relative;              //    position: relative;
  border: 1px solid #ccc;          //   border: 1px solid #ccc;
  padding: 10px;                   //    padding: 10px;
}                                  //  }

// Shorthand alias
@include B(notification) { ... }
```

----

#### `@include Mod($mod) { ... }`

Unrolls into a BEM block-modifier selector.

May be either:
- String: Simple modifier string. `mod` results in `.block--mod`
- Map: Modifier string with value. `(mod: value)` results in `.block--mod-value`

Notes:
- This mixin does not generate element-modifiers. Use `Elem(elem, $mod:mod)`.
- Nesting a Mod inside a pseudo-selector is not supported, because what that should mean isn't clear.

```scss
@include Block(notification) {       //  .notification--error {
  // ...                             //    background: rgba(255, 0, 0, 0.15);
  @include Mod(error) {              //    border-color: red;
    background: rgba(red, 0.15);     //  }
    border-color: red;               
  }                                  
                                     
  @include Mod((theme: growl)) {     //  .notification--theme-growl {
    top: 0;                          //    top: 0;
    right: 0;                        //    right: 0;
    width: 200px;                    //    width: 200px;
    height: auto;                    //    height: auto;
  }                                  //  }
                                     
  @include Mod((theme: bar)) {       //  .notification--theme-bar {
    bottom: 0;                       //    bottom: 0;
    left: 0;                         //    left: 0;
    width: 100%;                     //    width: 100%;
    height: 200px;                   //    height: 200px;
  }                                  //  }
}

// Shorthand alias
@include M(error) { ... }
```

----

#### `@include Elem($elem-name, $mod:null)`

Unrolls into a proper BEM element selector, depending on the context:
- Inside just a block, yields a root-level `.block__elem`
- Inside a mod or pseudo-selector, yields a nested `.block--mod .block__elem` 

If `$mod` is included, it is appended to the block selector, like `.block__elem--mod`. Multiple `$mod`s are not supported.

```scss
@include Block(notification) {                   //  .notification__dismiss-btn {
  // ...                                         //    position: absolute;
  @include Elem(dismiss-btn) {                   //    right: 0;
    position: absolute;                          //    font-weight: bold;
    right: 0;                                    //  }
    font-weight: bold;                           
  }                                              
                                                 
  @include Elem(dismiss-btn, $mod:on-left) {     //  .notification__dismiss-btn--on-left {
    left: 0;                                     //    left: 0;
    right: auto;                                 //    right: auto;
  }                                              //  }
                                                 
  // Works inside mods:                          
  @include Mod(error) {                          //  .notification--error .notification__dismiss-btn {
    @include Elem(dismiss-btn) {                 //    display: none;
      display: none;                             //  }
    }                                            
  }                                              
                                                 
  // ... and pseudo-selectors
  &:hover {                                      //  .notification:hover .notification__dismiss-btn {
    @include Elem(dismiss-btn) {                 //    transform: scale(1.1);
      transform: scale(1.1);                     //  }
    }
  }
}

// Shorthand aliases (mixin and argument)
@include E(dismiss-btn, $m:on-left) { ... }
```

-----

#### `@include BEM($block, $elem:null, $mods:null)`

Unrolls into a full BEM selector. See the `bem-selector` documentation for parameter details.

```scss
@include BEM(notification, $mod:error, $elem:dismiss-btn) {
  color: red;
}

//  .notification--error .notification__dismiss-btn {
//    color: red;
//  }
```

-----

### Functions

#### `bem-selector($block, $elem:null, $mods:null)`

Generates a full BEM selector.
- `$block`: Required. A string block name.
- `$elem`: Optional. A sub-element name. If `$mod` is not present, it is joined with `$block`. If `$mod` is present, it is nested under `$block--$mod`
May be either:
  - String: Simple element name. Results in `.block__elem`
  - List: Pair like `($elem-name, $elem-mod)`<br>
    Applies the element-modifier to the subelement. Multiple elem-mods are not supported. Same types as `$mod`.
- `$mod`: Optional. A block modifier. May be either:
  - String: Simple modifier string. Results in `.block--mod`
  - Map: ($modifier-name: $modifier-value). Results in `.block--mod-val`

- `$mods`: Optional. List. Allows generating selectors for multiple mods at once. Each element of the list should be one of the allowed type for `$mod`

```scss
$s: bem-selector(block);                   //  .block

$s: bem-selector(block, $e:elem);          //  .block__elem
$s: bem-selector(block, $e:(elem,emod);    //  .block__elem--emod

$s: bem-selector(block, $m:mod);           //  .block--mod
$s: bem-selector(block, $m:(mod:val));     //  .block--mod-val
 
$s: bem-selector(block, $m:mod, $e:elem);  //  .block--mod .block__elem

$s: bem-selector(block, $m:(mod-a,(mod-b:bval)), $e:elem);
// .block--mod-a .block__elem, .block--mod-b-bval .block__elem
```

You could use it like...

```scss
#{bem-selector(block, $e:elem, $m:mod)} { 
  // this is exactly what the BEM mixin does
}
```


## Shorthand Aliases

The provided mixins come with "shorthand aliases" so you can save some typing.

```scss
@include Block(b) { ... }
@include B(b) { ... }

@include Elem(e, $mod:m) { ... }
@include E(e, $m:m) { ... }

@include Mod(m) { ... }
@include M(m) { ... }
```


## Development

### One-time setup
```
npm install; npm install -g mocha;
```

### Run tests

```
npm test
```

Runs tests located in `/test`

## With love to...

- [Sass](http://sass-lang.com/) in general (duh - sass rocks)
- [libsass](https://github.com/sass/libsass) specifically (speed rocks)
- [true](https://github.com/oddbird/true) (tests rock)
- [SassMeister](http://www.sassmeister.com/) (instant feedback rocks, too)
- You (also rock)

## With love from...

[Rafflecopter](https://github.com/Rafflecopter)

## License

Released under [MIT license](https://opensource.org/licenses/MIT), same as Sass.

## TODO
- Provide easier installation & plugins for common build tools
- test _all_ functions. Right now at least `bem--extract-block` needs tests.
- try to test mixins. Might be tricky since they use `@at-root`
- automated test in various sass implementations?
