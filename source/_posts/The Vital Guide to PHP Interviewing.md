---
title: php面试指南
tags: php-devel
categories: php-devel
abbrlink: 7a743d7a
date: 2018-03-01 20:07:03
updated: 2018-03-01 20:07:03
---


Ubiquitous…that is definitely one word you could use to describe PHP in relation to the web. It really is everywhere. In terms of server-side programming languages, it is by far the most widely used, powering more than 80% of today’s websites (with the next runner up, ASP.NET, trailing way behind at a mere 17%).

Why? What makes PHP so popular and widely-used? While there’s no single answer to this question, PHP’s ease of use is certainly a significant contributing factor. PHP newbies can get up-to-speed and build dynamic PHP-based content into their web sites with a minimum of programming expertise required.

But therein lies much of the challenge of finding highly-skilled PHP developers. PHP’s relatively low barrier-to-entry and 20 year history means that PHP developers have become practically as ubiquitous as the technology itself. Yet while many can legitimately claim to “know” PHP, those who are true experts in the language are capable of producing software that is much more scalable, functional, robust, and maintainable.

So how do you distinguish those who have real competency in PHP from those with just a cursory knowledge, let alone those who are in the top 1% of candidates?

Toward that goal, this guide offers a sampling of effective questions to help evaluate the breadth and depth of a candidate’s mastery of PHP. It is important to bear in mind, though, that these sample questions are intended merely as a guide. Not every “A” candidate worth hiring will be able to properly answer them all, nor does answering them all guarantee an “A” candidate. At the end of the day, hiring remains as much of an art as it does a science (see In Search of the Elite Few).

Note that we have tried to keep this guide focused on modern PHP (i.e., versions 5.3 and beyond), but there are references to concepts and functions that have been around for a long time and should be familiar to any qualified PHP developer.
Key PHP Concepts and Paradigms

There are a number of key concepts and paradigms that are essential for a PHP expert to be well-versed in. Here are some examples.

Q: Describe a closure in PHP. Provide examples of when, why, and how they can be used.

Closures become useful when some piece of logic needs to be performed in a limited scope but retain the ability to interact with the environment external to that scope. The first building block of a closure is the anonymous, or lambda style, function which has no name associated with it. For example:

# The 2nd argument to array_walk is an anonymous function
array_walk($array, function($dog) {
    echo $dog->bark();
});

Although they themselves have no name associated with them, anonymous functions can be assigned to a variable or passed around as callbacks by higher order functions that require them. For example:

# Define an anonymous function and assign it to a variable
# named $dogs_bark.
$dogs_bark = function($dog) {  
    echo $dog->bark();
}
array_walk($array, $dogs_bark);

Under the hood in PHP, anonymous functions are implemented using the Closure class.

The contents of an anonymous function exist within their own scope, independent of the scope in which they were created. However, it is possible to explicitly bind one or more variables from the external scope to be referenceable within the anonymous function via the use clause in the function definition. Let’s illustrate:

class Dog {
    public function bark() { echo 'woof'; }
}

$dogs_bark = function($dog) use (&$collar) { # bind by reference
    if ($collar == 'fitsWell'){
         echo $dog->bark(); # 'woof'
    } else {
         echo 'no bark';    # collar is too tight to bark
    }
};

$dog = new Dog;

$collar = 'fitsWell'; # external variable
$dogs_bark($dog); # 'woof'

$collar = 'tight';
$dogs_bark($dog); # 'no bark'

This ability to access these external variables within a closure becomes particularly useful when using higher order functions. Take for example our array_walk usage from above which, like other functions of this type, operates in a very specific way on the subject variables which it is passed. As the function iterates over the $array, only the current value and key are passed to the anonymous function callback. There is no opportunity to pass in the $collar variable without the closure and it’s use clause . We could use the global keyword, but would needlessly pollute the global namespace with a variable that only makes sense in our very limited scope.

Closures have additional object oriented uses as well. PHP 5.4 brings new methods to the Closure class’ interface. Specifically, the new bind and bindTo methods can be used to bind to new objects for the closure to operate on, e.g.:

Closure::bindTo($newthis, $newscope);

This method will actually duplicate the closure itself and bind its scope to that new object, so $this actually references $newthis in the object context. To help illustrate, let’s modify the $dogs_bark function to use $this and then bind it to a new Dog object:

# define the closure, but not bound yet to any object
$dogs_bark = function() {
    echo $this->sound;  # where this is a property
};
$new_dog = new Dog();
# now create a new closure bound to the $new_dog object
$new_closure = $dogs_bark->bindTo($new_dog);
$new_closure();   # echoes the sound

The fact that a closure can be assigned to a variable and now has access to $this is quite powerful. In particular, this means that it can become a callable property of another object and essentially become a method of that object. For example:

$dog = new Dog();
$dog->closure = $dogs_bark;
$dog->closure();

This makes it possible to change and add to the behavioral capabilities of an object at runtime without the need to alter the defining class signature. Very useful when functionality enhancements are needed, but the actual code cannot be touched or where the need for the enhanced functionality is limited in scope.

Q: Explain the use and purpose of the global keyword in PHP. Provide an example of a case where its use would be appropriate, as well as one where it would not be.

In the days of “PHP gone by”, the language’s object oriented implementation was much less sophisticated than it is today. It is therefore not uncommon to find older “legacy” PHP code that makes fairly extensive use of the global keyword. In many ways, overuse of global variables “flies in the face” of modern object-oriented programming (OOP) best practices, as it may lead to intertwined dependencies between classes, difficulty decoupling disparate units of logic from one another, and pollution of the global namespace with variables that have no (or very limited) use in that context.

Consider the following simple example:

class Dog {
    function bark() {
        global $sounds;
        return $sounds->bark();
    }
}

This introduces a hidden dependency of the Dog class on the global $sounds object. While there may be cases where this would be justified (e.g., a system with one single well-defined set of sounds that is never modified), generally speaking, it would be much better to explicitly pass the relevant $sounds object to the Dog class’ constructor, which would then be stored and used within that instance of the class; e.g.:

class Dog {
    protected $sounds;
    function __construct($sounds) {
        $this->sounds = $sounds;
    }

    public function bark() {
        return $this->getSounds()->bark();
    }

    public function getSounds() {
        return $this->sounds;
    }
}

But as is true with most things in coding, never say never. There are, in fact, a number of robust and stable products out there written in PHP that make heavy use of globals. The Wordpress Codex, which powers roughly 20% of the websites on the web (and rising yearly), Joomla! 1.5 (which is still in wide use), and the iLance Enterprise Auction system are all examples of projects that have been around for years and make use of ‘mature’ globals. Similarly, there may be situations where the use of globals in your code is appropriate, but their use should always be approach with caution.

Q: Describe namespacing in PHP and why it is useful.

One of the newer additions to PHP’s support for OOP is the namespace. As the name implies, a namespace defines a scope in a program where class, interface, function, variable, and constant definitions won’t produce name collisions with similarly named items in other namespaces.

Prior to PHP 5.4, class names would often become quite long in an attempt to match the package hierarchy and avoid name collisions. For example, let’s say we have a Dogs class defined in the model of the Dog_Pound application. Without namespacing, we might feel compelled to provide a verbose name for our class such as the following:

class Dog_Pound_Model_Dogs {  # wow, that's a mouthful!
    function getDogs();
}

Namespacing helps alleviate this issue by enabling the developer to explicitly specify the namespace within which all named items (classes, variables, etc.) are to be defined. With namespacing, the above verbose class name could be replaced with:

namespace dog_pound\model;  # specify the current namespace

class Dogs {   # only "Dogs" defined in dog_pound\model
    function getDogs();
}

$dogs = new Dogs; # unambiguous since only one in this namespace

Relative namespace references are supported as well. For example, when in the dog_pound namespace, you can use the following to instantiate your class:

$dogs = new model\Dogs  # only one "model" defined in dog_pound

In addition, named items in other namespaces can be imported in other files and can then be referenced directly. This is done via the use operator as follows:

namespace this\new\namespace;  # our current namespace

# import Dogs from dog_pound\model namespace and reference it
use dog_pound\model\Dogs;

$dogs = new Dogs;

Q: What are traits? Describe their major characteristics and why they are useful. Give an example of a trait declaration and a class that uses multiple traits.

Traits are an excellent addition to PHP 5.4 that allow behaviors to be added to a class, without needing to extend a parent class to inherit the desired functionality (prior to PHP 5.4, this could only be done via a mixin class pattern at runtime). Additionally, you can make use of multiple Traits in a single class. This makes them a powerful aid in the organization and separation of concerns in the codebase and, as such, helps to honor the composition over inheritance design principle.

Here are a couple of simple trait definition examples:

trait Movement {
    public function topSpeed() {
        $this->speed = 100;
        echo "Running at 100 %!" . PHP_EOL;
    }
    public function stop() {
        $this->speed = 0;
        echo "Stopped moving!" . PHP_EOL;
    }
}

trait Speak {
    public function makeSound(){
        echo $this->sound . PHP_EOL;
    }
}

The use keyword is once again employed, this time for importing from another namespace to include one or more traits in a class definition. For example:

class Dog {
    use Movement, Speak;  // Dog now has these capabilities!

    protected $sound;

    function __construct() {
        $this->sound = 'bark';
    }
}

$dog = new Dog();
$dog->topSpeed();  // from Movement trait
$dog->stop();      // from Movement trait
$dog->makeSound(); // from Speak trait

PHP at Your Fingertips

Committing to memory what can easily be found in a language specification or API document is no indicator of proficiency. But that said, someone who is intimately familiar with a programming language will have its syntactical and functional details and nuances at his or her fingertips. Here are some questions that help evaluate this dimension of a candidate’s expertise.

Q: Describe the relationship between php://input and $_POST. How would you access the php://input stream?

In the simplest terms, the $_POST superglobal is the formatted or parsed content of the body of a request made to the server with the post method.

POST /php-hiring-guide/php-post.php HTTP/1.1
Host: toptal.com
Referer: http:///toptal.php/php-hiring-guide/php.php
Content-Type: application/x-www-form-urlencoded
Content-Length: 63
article_name=PHP Hiring Guide&tag_line=You are hired!&action=Submit

The body of the request can be accessed through the PHP’s input stream the same as any other file:

$input = file_get_contents("php://input");

Q: Name and define at least five superglobals that begin with $_. Describe their relationship to the $GLOBALS variable.

    $_POST - a hash of key value pairs sent to the server with the ‘post’ method
    $_GET - a hash of key value pairs sent to the server with the ‘get’ method
    $_REQUEST - an amalgamation of $_GET and $_POST
    $_SERVER - a hash of variables set specifically by the web server, and relevant to the execution of the program
    $_ENV - a hash of variables related to the host machine and it’s configuration
    $_SESSION - a hash of variables that are meant to be persisted between page views or separate application executions
    $_COOKIE - a hash of variables that are to be stored with the client
    $_FILES - a special type of request hash that holds input data related to an uploaded set of files

Akin to the global keyword in PHP is the $GLOBALS super global variable. As the name suggests, superglobals are automatic global variables and as such are stored in $GLOBALS. For example, the $_ENV superglobal could alternatively be accessed via $GLOBALS["_ENV"].

Q: Explain the purpose and usage of the __get, __set, __isset, __unset, __call, and __callStatic “magic” methods. When, how, and why (and perhaps why not) should each be used?

The first four methods in our list, __get, __set, __isset, and __unset are used for property overloading of an object. They let us define how the outside world is able to interact with properties that have a private or protected visibility, and properties that don’t even exist in our object.

# 'whiskers' is not defined in the Dog class but is handled by
# the Dog class' __get method as follows:
function __get($name) {
    if ($name == 'whiskers') {
        # only create one whiskersService per instance
        if (! isset($this->whiskersService)) {
            $this->whiskersService = new Whiskers($this);          
        }
        # so calls to 'whiskers' will load the wiskersService
        return  $this->whiskersService->load();
    }
}

A caller can then simply retrieve the whiskers property like so:

$hairs = $dog->whiskers;

By using the __get method to “intercept” this reference to a seemingly public property we are able to obscure the implementation details of one or more of the object’s properties.

Use of the __set method is similar. For example:

function __set($name, $value) {
    if ($name == 'whiskers') {
        if ($value instanceOf Whisker) {
            $this->whiskersService->setData($value);
            return $this->whiskersService->getData();
        } else {
            throw new WhiskerException("That's not a whisker");
            return false;
        }
    }       
}

A caller can then simply set the whiskers property like so:

$dog->whiskers = $hairs;

This statement automatically invokes the __set method, using 'whiskers' as its first argument and the right side of the assignment as the second argument.

Lastly, the __isset and __unset methods complete the quartet. They each take just one argument, the $name of the property to be evaluated or operated on; e.g.:

function __isset($name) {
    if ($name == 'whiskers') {
        return (bool) is_object($this->whiskersService) &&
                                !empty($this->whiskersService->getData());
    }
}

function __unset($name) {
    if ($name == 'whiskers' && is_object($this->whiskersService)) {
        # we don't want to completely kill the service:
        $this->whiskersService->reset();
    }
}

The other two methods, __call and __callStatic, perform a similar function for classes, but give us method overloading. Through their use we can define how a class should react when an undefined, protected, or private method is called.

In a __call implementation where we want to ensure all “non-visible” method calls get returned the payload from the whiskersService we might do something like:

public function __call($method, $args) {
    return $this->whiskersService->load();
}

__callStatic has the same arguments, the same functionality and allow for this same interaction, but outside of a specific object context. This means that if we have this method defined we can use the ClassName::notVisibleMethod() syntax; e.g.:

public function __callStatic($method, $args) {
    if (!is_object(static::$whiskersService)) {
        static::$whiskersService = new Whiskers(__CLASS__);
    }
    return static::$whiskersService->load();
}

$hairs = Dog::whiskers();

In the above set of examples, we have encapsulated the whiskers implementation from the outside world and made it the only available object exposed in this way. The clients of such properties and methods do not need to know anything about the underlying whiskersService or how the Dogs class is storing its data at all.

Collectively, these methods increase our ability to flexibly compose objects. The opportunity for object abstraction and encapsulation, and thus reusable, more compact code (and ultimately a more manageable system as a result) are also increased.

Care should be taken, though, in using these convenient methods, as the benefits can come at a cost. They are slower than straight access to an otherwise public property in question, and also slower than defined getters and setters. They also hamper use of certain useful capabilities (such as object reflection, autocomplete in your IDE, automatic documentation utilities like PHPDocumentor, and so on). So if these are facilities you want or need to rely on, you may consider defining methods and properties explicitly instead. As with most things, there’s no across-the-board right answer. The pros and cons should be evaluated on a case-by-case basis.

Q: Describe one or more Standard PHP Library (SPL) data structures. Give usage examples.

Most of PHP development deals with getting and processing data from one source or another, such as a local database, local files, a remote API, etc. As such, developers spend a great deal of time getting, organizing, moving, and manipulating that data. In some cases, an array just won’t cut it in terms of memory usage and performance and therefore better data structures are called for.

Also, with so much talk and focus on frameworks (this page could be filled with names of PHP frameworks!), it could be difficult to find a developer who has a great deal of experience with the particular framework that your project is using. The net effect of a such an active development community is that the distribution of developers working with a particular framework starts to shrink, and could make it harder for you to find that specific skill.

One area to focus on, and where you be reasonably sure that a PHP expert will have familiarity, is in the use of and general makeup of the Standard PHP Library (SPL). If the candidate has a solid background here, the chances of success for that candidate working into and adding value to your application are greater, regardless of the specific frameworks being used in your environment.

Specifically, a candidate should be familiar with some or all of the nine SPL data structures listed in the PHP Manual. Many provide highly similar functionality to one another, but the slight variations make each one more perfectly suited to a particular use. Here’s a brief overview:

    SplDoublyLinkedList. Each element in this list holds links to the node before it and the node after it in the list. Picture that you are on line at the bank, but you are only able to see the person in front and the person behind you. That is analogous to the ‘link’ relationship between elements in a SplDoublyLinkedList. Inserting an element into the list is akin to someone cutting in front of you at the bank (but you then suddenly forget who was in front of you, and that person also forgets you entirely). Enables efficient transversal through a dataset and lets you list and add large amounts of data, with no internal need to re-hash.
    SplQueue and SplStack. Very similar to the SplDoublyLinkedList are the SplQueue and SplStack. Both of these are essentially SplDoublyLinkedLists with different iterator flags (IT_MODE_LIFO is “Last In First Out” and IT_MODE_FIFO is “First In First Out”), which govern which order the elements are read in and what to do with those elements after they have been processed. One other distinction is that the SplQueue API might be considered a bit more intuitive, supplying an enqueue() method (rather than push()) and a dequeue() method (rather than shift()).
    SplHeap. Represented internally as a binary tree, where each node in the tree has a maximum of two child nodes. It is an abstract class which must be extended to define a compare() method. This method is then used to perform real time sorting whenever a value is inserted into the tree.
    SplMaxHeap and SplMinHeap. Concrete implementations of the SplHeap abstract class. SplMaxHeap provides a compare() method that maintains values in order from highest to lowest, whereas SplMinHeap provides a compare() method that maintains values in order from lowest to highest.
    SplPriorityQueue. Similar to an SplHeap, but sorting is done based on an additional “priority” value supplied for each member.
    SplFixedArray. Similar to a regular array, but indices can only be integers and the length of the array is fixed. It provides improved speed when implementing an array. There is no processing speed benefit over an array, the interface is optimized so that the objects don’t need to be hashed manually when adding elements.
    SplObjectStorage. Provides an interface for mapping objects to data or simply providing a container for a set of objects. Essentially, it can use an object much like an associative array key to relate that object to some data or to no data at all.

Q: What will $x be equal to after the statement $x = 3 + "15%" + "$25"?

The correct answer is 18. Now let’s explain why.

PHP supports automatic type conversion based on the context in which a variable or value is being used.

If you perform an arithmetic operation on an expression that contains a string, that string will be interpreted as the appropriate numeric type for the purposes of evaluating the expression. So, if the string begins with one or more numeric characters, the remainder of the string (if any) will be ignored and the numeric value is interpreted as the appropriate numeric type. On the other hand, if the string begins with a non-numeric character, then it will evaluate to zero.

With that understanding, we can see that "15%" evaluates to the numeric value 15 and "$25" evaluates to the numeric value zero, which explains why the result of the statement $x = 3 + "15%" + "$25" is 18 (i.e., 3 + 15 + 0).

Depending on who you ask, automatic type conversion is either a great feature or one of the worst pitfalls of the PHP language. On the one hand, when used consciously and properly, it can be very convenient for the developer (e.g., I don’t need to write any code to convert “15” to 15 before using it in an arithmetic operation). On the other hand, it can easily lead to headfakes and hard-to-track-down bugs when it is employed inadvertently, since no error message or warning will be generated.

Q: Explain the significance of using the static keyword when invoking a method or property, including how it differs from the self keyword? When would you use it and why? Provide an example.

As of PHP 5.3.0, PHP implements a feature called late static bindings. “Late binding” comes from the fact that static:: will not be resolved using the class where the method is defined but will rather be determined using runtime information and scope. (Incidentally, the term “static” is a bit of a misnomer in that this is not limited to being used with static methods.)

Consider the following example:

class Dog {
    static $whoami = 'dog';
    static $sound = 'bark';

    function makeSounds() {
        echo self::makeSound() . ', ';
        echo static::makeSound() . PHP_EOL;
    }

    function makeSound() {
        echo static::$whoami . ' ' . static::$sound;
    }
}

Dog::makeSounds();

Unsurprisingly, this will output “dog bark, dog bark”.

But now let’s extend this example to include the following:

class Puppy extends Dog {
    static $whoami = 'puppy';
    static $sound = 'woof';

    function makeSound(){
        echo static::$whoami . ' whimper';
    }
}

Puppy::makeSounds();

You may be surprised to learn that this will output “puppy woof, puppy whimper”. To understand why, let’s revisit the makeSounds() method in the Dog class.

The makeSounds() method in the Dog class first invokes self::makeSound(). self:: instructs PHP to always use the scope of the same class (in this case, the Dog class). Therefore, calling self::makeSound() from within Dog’s makeSounds() method will always invoke Dog’s version of makeSound(). This is true even when we call Puppy::makeSounds() since Puppy has no makeSounds() method of its own and instead relies on Dog’s version. This is why the call from Puppy::makeSounds() to self::makeSound() outputs “puppy woof”.

Next, the makeSounds() method in the Dog class invokes static::makeSound(). static:: behaves differently from self::. static:: instructs PHP to use the version in the scope of the class that invoked the method at runtime (in this case, the Puppy class). Hence, when the call to static::makeSound() is made, it invokes Puppy’s version of that method. This is why the call from Puppy::makeSounds() to static::makeSound() outputs “puppy whimper”.
PHP Under-the-Hood

Understanding how PHP works “under the hood” is one of the primary defining characteristics of a PHP expert. Such a candidate will understand not only how to do something, but the various options available, and the performance and functional ramifications of each.

Q: How does PHP build an array internally?

Central to much of PHP development is the array. These are good for most uses where an iterable data structure is needed. Internally in PHP, arrays are stored, like many other data structures, as hash tables. PHP is written in C where there is no such thing as an associative array: arrays in C can only have integer indices. Therefore, a hash function is used to translate PHP array indices (both integer and string) into appropriate numeric indices and the array values are stored at that location or bucket in the hash table.

The process of finding bucket entries based on a hash of the array keys is of O(1) complexity (using Big O notation). This because no iteration over the hash is needed, since the hash function produces the exact location in the table. (Note that this changes slightly if there is more than one element in the bucket.)

PHP automatically increases the size of an array “on demand”. Specifically, when an element is added to an array, if addition of the element would cause the array to exceed its currently allocated size, PHP responds by doubling the memory allocation. The corresponding hash table needs to be completely rehashed as a result.

Let’s consider what that means for just a second: Adding values to an array can in some cases be as complex as iterating over that entire new array, and in that same situation you have doubled the memory footprint of that structure. Given this, where datasets begin to get large and memory is an issue, it may be wise to consider an alternative to an array, where loading the data for processing happens on a more as needed basis.

Q: What is the difference between the ArrayAccess and ArrayObject?

ArrayAccess is simply an interface definition that requires any class that implements it to define the following methods: offsetGet, offsetSet, offsetExists, and offsetUnset.

ArrayObject, on the other hand, is an instantiable class that implements the ArrayAccess interface. A nice usage convenience of an ArrayObject is that its elements can either be accessed the same as a standard array (i.e., via the familiar [] notation; e.g., $dogs['sound']) or via the offsetGet(propertyName) method (e.g., $dogs->offsetGet('sound')).

Q: What is a generator? When might you use it instead of an iterator or a plain array. What do yield and send do?

A generator looks much like any other function in PHP and shares some similarities with closures. When called for the first time in an iterable context (e.g., a foreach statement), the generator returns an object that is an instance of the Generator class, which in turn implements the Iterator interface. The values for iteration are generated in real-time eliminating the need to load an entire data set into memory before hand. That means that the result can be iterated over in much the same way that an array can, but with some important differences; namely they:

    Can only be used in a forward run, meaning that one can’t iterate backwards over the values or start at the beginning. Calling the rewind() method on the generator will throw an exception.
    Are slower than processing a pre-populated array (the generator is doing extra processing to produce the value).
    Are extremely memory efficient.

One of the chief differences between a generator function and other PHP functions or closures is that they don’t make use of return to provide their results. Instead, yield is used to pass values out of the generator (e.g., yield $dog;). In fact, specifying a value on a return statement in a generator will produce a parser error, but an empty return statement will simply stop the generator altogether.

yield pauses execution and returns the current value to the scope of the structure that is using the generator. That structure has the opportunity to send information back to the generator at that same point in its execution via the send method (e.g., $generator->send(-1);).

With that basic explanation, let’s take a look at a simple example of a generator’s use.

First, let’s define our generator:

function dog_generator() {
    foreach (range(0, 2) as $value) {
        # some fake data source
        $dog = DogHelper::getDogFromDataSource($value);
        # catch input from send or use $dog
        $input = (yield $dog);    
        if ($input == -1) {
            return; // stop the generator
        }
    }
}

Note that, in the above example, we are assigning the result of the yield method call to a variable and must therefore wrap the yield in parentheses in order for it to be syntactically valid.

And now here’s an example of how we would use the above generator:

# Get an instance of the generator and assign it to a variable
$generator = dog_generator();

foreach ($generator as $dog) { # $dog is a result of 'yield'
    echo $dog . PHP_EOL;
    // we wanted to find the terrier
    if ($dog == 'Terrier') {
        # send an input into the generator via 'yield'
        $generator->send(-1);
        echo 'We found the Terrier, stop the show.';
    }
}

When we run the above code our example stops when it finds “Terrier”. For example:

Dalmatian
Terrier
We found the Terrier, stop the show.

The key benefits of a generator are ease of implementation, increased readability of code, and overall improved performance and memory usage.

So generally speaking, it’s best to use a generator where there is no need to rewind or iterate back over the values again in another part of the program. On the other hand, if memory is not a concern, then a simple array of the values is much faster.
The Big Picture

Q: List some of the major differences between (or sequential additions and improvements to) PHP 5.3, PHP 5.4 and PHP 5.5.

PHP 5.3 was released back in June 2009 but is still in wide circulation. Key new features in this release included: closures, lambda style functions, namespacing, and late static binding.

PHP 5.4 was released in March 2012, introducing Traits, as well as the ability to declare an array with a shortened [] syntax (e.g., $array = ['shih tzu', 'rottweiler']). In addition, register_globals was removed in this version.

PHP 5.5 was released in June 2013 and is the current release. This version adds support for generators (and their related yield and send keywords), as well as adding finally to the try/catch exception handling syntax. APC Cache is gone in favor of OpCache (based on the Zend Optimizer). PHP 5.5 also takes the array literal even further and supports syntax such as ['shih tzu', 'rottweiler'][1] (which returns “rottweiler”) as well as the handy notation of strings interpreted as arrays (e.g., '6, 5, 4'[0], which returns “6”).
Wrap-up

Finding PHP developers can be relatively easy, but finding stellar PHP developers is a formidable challenge. The questions presented in this guide can serve as useful tools in your overall recruiting toolbox to better identify those who have mastered the language. Finding such candidates is well worth the effort, as they will undoubtedly have a significant positive impact on your team’s productivity and results.
