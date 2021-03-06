[Home](../readme.md "Home")

#### SECTION 1
----
## AbstractObject

### add() method

While you might have used “new” operator before to create class instances, in Agile Toolkit you will use the operation called **<u>Adding</u>**.

    $new_object = $owner->add('MyClass');

While the syntax is somewhat similar to the “new” operator, the fundamental difference is that the $new_object is created within the scope of it's $owner.

Depending on the class of the parent and child objects, there might be different effects. If you add **<u>Button</u>** into **<u>Page</u>** then sub-sequential rendering of a **<u>Page</u>** will also render a **<u>Button</u>**.

Newly created objects are also well aware of their owner-object:

    $new_object -> owner === $owner; // knows who added it

### The top-most Object

You might be curious. If you need an object to add another object, where is the top-most object coming from? The top-most object is called “**<u>Application</u>**” and it is initialized first.

Below is a sample command-line application in Agile Toolkit:

    include 'atk4/loader.php';
    $api = new ApiCLI();    // application object
    $my_object = $api->add('MyClass');
    $my_object->api === $api;    // for any object

All objects can access the top-most Application object through "api" property.

### Object's name

As you add more objects, each of them generates unique name. This name can be accessed though a "name" property. It is based on a parent object's name. A custom name can be specified as a second argument to add(), but if omitted, then the class name will be used instead.

    $new_object = $owner->add('MyClass','foo');
    echo $new_object->name;    // name_of_owner_foo
    echo $new_object->short_name; // foo
    
Unique names are important because that's how objects can identify themselves. This is often used in AJAX requests, HTML ID properties and selective rendering of HTML.

Note: you may change name of an object by calling rename(), but it must used with caution as it may have some side-effects on a complex objects.

### Element Tracking

When adding an object, the owner will create a note inside it's “elements” property about the new object. This process is called element tracking. When you are adding one view into another, this connection will help to perform recursive rendering without you explicitly calling render of child objects. However the default behavior for non-view objects is not to keep track of elements.

The “elements” property of non-views would contain 'shor_name'=>true; association. Therefore if you loose the reference to your object (which is returned by add()) the object would be destroyed.

This helps Agile Toolkit to get rid of models, controllers when you no longer need them through the use of PHP garbage controller. There are however some exceptions, where tracking is necessary, such as Model Fields which needs to be associated with the Model object.

### Method init()

When you add an object, Agile Toolkit will automatically execute method init() of a new object. This method is called after “owner”, “api” and “name” properties are set.

Below is the example from Strength Checker indicator:

    class StrengthChecker extends View {
        function init(){
            parent::init();
            if(!$this->owner instance_of Form_Field_Password)
                throw $this
                ->exception('Must be added into Password field');
            // ....
        }
    }
    
You will find that init() method can be redefined in ANY objects in Agile Toolkit. The fundamental principle of writing your own init() method is:

* Avoid heavy-lifting. Do that in render() method instead, if possible.

----

**<center>Addition of objects in Agile Toolkit is referred to by Object-Oriented developers as "Incapsulating". Often functionality of $this object is extended by a new object.</center>**

----

### Return Value of add()

A call to add() will always return a new object of specified class. You can rely on that and chain additional calls to a returned value.

    $page->add('Button')->setLabel('Click Me');
    
In a true jQuery spirit, most of the object methods will return reference to themselves ($this) so that you can do more chaining.

### Indirect Adding of Objects

For convenience, objects may define new methods for adding certain types of objects. For example, Form has method called "addField" and grid has a method addButton();

    $form->addButton('Click Me');
    // almost same as
    $form->add('Button')->setLabel('Click Me');

In reality those methods will do bit more than just calling add() for you.

### Dependency Injection / Properties

If you specify second argument to add() an array, it will be used to initialize properties of a new object before init() is called. CRUD::init() adds Grid, but if you want to make it use a different class you can do it by specifying second argument.

    $page->add('CRUD',array('grid_class'=>'MyGrid'))
        ->setModel('User');

You can still specify name through array with the key of 'name'

### Using namespaces / add-ons

Agile Toolkit allows you to add objects from add-ons. Each add-on resides in it's own namespace, so to add object you should use slash to separate namespace and class name:

    $page->add('myaddon/MyClass');
    
Note: as you move code from your core app to add-on the format of using add() from within a namespace remains the same.

### Working with Existing Objects

AbstractObject provides you with several handy methods to access objects you added into it.

    $view = $page->add('View','myview');

    $page->hasElement('myview'); // returns $view or false

    $page->getElement('myview'); // returns $view or exception

    $view->destroy(); // removes object from parent and memory

### Cloning and newInstance()

Models and Controllers of Agile Toolkit are generally OK to clone, however there is an alternative way: newInstance(). This method will add a new object of a same class added into the same parent.

If you experience problems with clone, try newInstance().

### Adding Views

When you add a View into another View, you can specify two extra arguments for add() method. Third argument define where exactly HTML will appear and the fourth argument can override the template used by a View. I will further look into this when I'll be reviewing the chapter dedicated to Views.

### String presentation

You can convert objects to string:

    echo $page->add('View');
    // outputs Object View(name_of_view)

### Adding object

If object is already created, you can use it as a first argument of an add() method. This will re-assign the object's owner property. Please use this functionality with caution.

    $form = $page->add('Form');
    $field = $form->addField('line','foo');
    $frame = $form->add('View_Frame');
    $frame->add($field); // moves field inside frame

### Model property

For any object you can call setModel() method. It will initialize the model if necessary and will assign it to the "model" property. Although it's easier to simply access the model property, you can also use getModel() method. You can specify object to setModel();

    $grid = $page->add('Grid');
    $form = $page->add('Form');
    
    $grid->setModel('User'); // uses class Model_User
    $form->setModel($grid->model); // uses same object
    
Many objects redefine setModel() to perform additional binding of the model. When calling setModel() on a View, you can specify second argument, which is list of **<u>actual fields</u>**.

### Using setController

You can use setController as a shortcut for add('Controller_...'). Again, you may use either a controller by name or an object. The name will be prefixed with 'Controller_'.

### Session management

All objects in Agile Toolkit come with four methods to access session data: memorize(), learn(), forget() and recall(). You can specify the “key”, which will be used in scope of a current object:

    $obj1 = $this->add('MyObject');
    $obj2 = $this->add('MyObject');
    
    $obj1->memorize('test','foo');
    
    $obj2->recall('test'); //returns null
    $obj1->recall('test'); // returns foo

You can learn more about Agile Toolkit management under the Application section.

| Method | Arguments | Description |
| --- | --- | --- |
| memorize | name, value | Store value in session under name |
| recall | name, default | If value for name was stored previously return it, otherwise return default |
| forget | name | Remove previously memorized value from session |
| learn | name, value1, value2, value3 | Memorize first non-null argument. |

### Exceptions

Agile Toolkit takes full advantage of exceptions. You can read more about exceptions in ...

To raise exception please use the exception() method of an object:

    $obj = $this->add('MyObject');
    throw $obj->exception('Something is Wrong');
    
    // alternatively, add more info to exception
    throw $obj->exception('Something is Wrong')
        ->addMoreInfo('foo',123);

My default objects through exception of a class "BaseException". You may use your own class:

    class Exception_MyTest extends BaseException {
        function init(){
            parent::init();
            $this->addMoreInfo('thrown by',$this->owner->name);
        }
    }
    throw $obj->exception('Something is Wrong','MyTest');

Default exception type can also be specified on per-object basis through a property:

    class Model_Test extends AbstractModel {
        public $default_exception='Exception_MyTest';
    }
    // or
    $this->add('MyObject',array('default_exception'=>
        'Exception_MyTest'));

Third argument to exception() can specify a default exception code which can further be accessed through "code" property.

### Hooks

Hooks is a callback implementation in Agile Toolkit. Hooks are defined throughout core classes and other controllers can use them to inject code.

For example:

    // Add gettext() support into the app
    $this->api->addHook('localizeString',function($obj,$str){
        return _($str);
    });
    
The localizeString hook is called by many different objects and through adding a hook you can intercept the calls.

    $obj = $this->add('MyClass');
    $obj->addHook('foo',function($o){ return 1; });
    $obj->addHook('foo',function($o){ return 2; });
    $res = $obj->hook('foo'); // array(1, 2);

This example demonstrates how multiple hooks are called and how they return values. You can use method breakHook to override return value.

    $obj = $this->add('MyClass');
    $obj->addHook('foo',function($o){ return 1; });
    $obj->addHook('foo',function($o){ $o->breakHook('bar'); });
    $res = $obj->hook('foo'); // 'bar';

You should have noticed that all the hook receive reference to $obj as a first argument. You can specify more arguments either through hook() or addHook() 

    $obj->addHook('foo',function($o,$a,$b,$c){
        return array($a,$b,$c);
    }, array(3));
    $res = $obj->hook('foo',array(1,2)); // array(array(1,2,3));

When calling addHook() the fourth argument is a priority. Default priority is 5, but by setting it lower you can have your hook called faster.

    $obj = $this->add('MyClass');
    $obj->addHook('foo',function($o){ return 1; });
    $obj->addHook('foo',function($o){ return 2; },3);
    $res = $obj->hook('foo'); // array(2, 1);

Note: in this example, the "3" was passed as 3rd argument not fourth. addHook automatically recognize non-array value as a priority if array with argments is omitted. **<u>Argument</u>** omission is often used in Agile Toolkit methods.

When you are building object and you wish to make it extensible, adding a few hooks is always a good thing to do. You can also check the presence of any hooks and turn off default functionality:

    function accountBlocked(){
        if(!$this->hook('accountBlocked'))
            $this->email('Your account have been blocked');
    }

Without any hooks, hook() will return empty array.

Finally you can call removeHook to remove all hooks form a spot.

    $obj = $this->add('MyClass');
    $obj->addHook('foo',function($o){ return 1; });
    $obj->removeHook('foo');
    $res = $obj->hook('foo'); // array();

Note: If your object implements handlers for a few hooks and sets them inside init(), then after cloning such an object, it will not have the handlers cloned along with the object. Use of newInstance() should work fine.

### Method Calling

PHP has a great way of extending object methods through a catch-all. Agile Toolkit implements catch-all for all of its objects which is then wrapped through some hooks to bring the functionality of addMethod(). It allows you to register method in any object dynamically. Note, that if such method already exists, you wouldn't be abel to register a method. This is typically used for compatibility, when some methods are obsoleted and controller re-adds those methods back.

    $obj = $this->add('MyClass');
    $obj->addMethod('sum',function($o,$a,$b){ return $a+$b; });

Agile Toolkit also adds hasMethod() into all object as a preferred way to check if method exists inside object;

    if($obj->hasMethod('sum')){
        $res = $obj->sum(2,3); // 5
    }

You can also remove dynamically added methods with removeMethod();

[Next Section](section2.md "Next Section")