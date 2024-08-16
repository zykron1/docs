Creating a Validator
====================

To create a custom validator, you can define a class that specifies the validation rules for each field. This is done using the `Validator` class from the `libmercury` library. Below is an example of how to create a `SigninValidator` for validating user sign-in data.

Example Validator Class
-----------------------

.. code-block:: python

    from libmercury import Validator 

    class SigninValidator:
        username = Validator.String(min=3, max=20)
        password = Validator.String(min=8, max=50)
        isActive = Validator.Boolean()
        age = Validator.Integer(min=13, max=101)
        pets = Validator.Array(min=0, max=3)

        @Validator.Object()
        class cars:
            name = Validator.String
            horsepower = Validator.Integer(min=0, max=3000)

This `SigninValidator` class defines several fields, each with its own validation criteria:

- **username**: A string with a minimum length of 3 characters and a maximum of 20.
- **password**: A string with a minimum length of 8 characters and a maximum of 50.
- **isActive**: A boolean value indicating whether the user is active.
- **age**: An integer between 13 and 101.
- **pets**: An array with a minimum of 0 and a maximum of 3 elements.
- **cars**: An embedded object containing two fields:
  - **name**: A string for the car's name.
  - **horsepower**: An integer between 0 and 3000 representing the car's horsepower.

This structure ensures that incoming data adheres to the specified rules before processing.
