# RedBean Model Validation Plugin
With this plugin you will be able to filter and validate your RedBean Model, to do that, the plugin itself uses **[GUMP](https://github.com/Wixel/GUMP)**, a standalone PHP data validation and filtering class, which means that this plugin requires it, but do not worry as Composer will handle that dependency.

## Installation
Install the plugin through composer:

`composer require filisko/redbean-validation-plugin`

Once you installed the plugin with composer, you have to load it into RedBean like this:
```php
R::ext('validate', function($bean){
    return Filisko\RedBeanPHP\Plugin\ModelValidation::validate($bean);
});
```

You will probably have something like that after adding the validation plugin;
```php
R::setup('mysql:host=localhost;dbname=my_database','root', 'password');
R::freeze(TRUE);
R::ext('validate', function($bean){
    return Filisko\RedBeanPHP\Plugin\ModelValidation::validate($bean);
});
```


## How to use
The thing that matters here is that RedBean must be able to "connect" the bean with this model. The mechanism in RedBeanPHP that connects beans to models is called FUSE, because beans are fused with their models. [Read more about RedBean Models](http://www.redbeanphp.com/index.php?p=/link_beans)

Now, the important thing here is to make sure that your Model **contains the rules**, which will be used by GUMP to do the work.

#### Available options for fields

**label**: This option is used to replace the field name with the label on errors, and instead of showing the field name, show the label.

**filter**: This option is used to apply any filter to the value of a specific field.

**validation**: This option is used to validate the value of a specific field.

Please have a look to the available validations and filters on [GUMP GitHub repository](https://github.com/Wixel/GUMP).

**message**: This option is used to use a custom error message, remember that you must be very clear as it will show this error for any validation failure.

### Examples

An example of a RedBean model and his rules. Remember that you can always use your own model, the **MOST IMPORTANT** thing is RedBean to be able to FUSE (connect) your Bean with your Model so the validation plugin can use the specified rules inside your Model using the Bean. It may sound a bit complicated but with the examples below you will see how easy it is.

**User.php**
```php
<?php
class User extends \RedBeanPHP\SimpleModel
{
    public static $rules = [
        'username' => [
            'label' => 'Username',
            'filter' => 'trim|sanitize_string',
            'validation' => 'required|alpha_numeric'
        ],
        'password' => [
            'label' => 'Password',
            'validation' => 'required|alpha_numeric|min_len,6'
        ],
    ];
}
```

Example of using the "user" Bean and validating it.
```php
<?php
$user = R::load('user', 1); //  Load user from database with ID 1
$user->username = 'this_is_my_username'; // Change the username
$validation = R::validate($user); // The magic
// If $validation does not return true, then we have some errors, and $validation will return an array of these errors
if ($validation !== true)
{
    foreach ($validation as $field=>$message)
    {
        echo $message;
        echo "<hr>";
    }
}
else
{
    // If everything went OK just save it
    R::store($user);
}
```
So the output of this example would be "The Username field may only contain alpha-numeric characters" as we are using underscores.

Also remember that if the validation passes, the value will be saved with the applied filters. You can create a GUMP filter like this:
```php
// Filter "upper" that will put the value in uppercase
\GUMP::add_filter("upper", function($value) {
    return strtoupper($value);
});
```
As I said before, I recommend you to have a look to [GUMP repository](https://github.com/Wixel/GUMP).

### Exceptions
The exception **ModelValidation_Exception** will be thrown if you try to validate a Bean that does not have a Model or validation rules.

