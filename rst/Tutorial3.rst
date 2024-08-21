Tutorial 3
==========
This tutorial will be based of the `previous tutorial <tutorial2.html>`_. Please complete it first.

In the last tutorial we covered how to use the built-in ORM capabilities. In this tutorial we will cover validators.
In mercury a validator is a preprocesser that can be attached to a controller that automatically filters out bad
requests. For example, you might want to make sure that you only recive valid requests to your API that. Lets say
you have a controller that takes in a username and a password. A validator would make sure that the username and
password are not empty and that the input is a string. This form of preprocessing can be easily achived with the CLI:

.. code-block:: bash

	python3 -m libmercury create validator Signin

Once you run this, we can open up the file that it created in src/validators/SigninValidator.py:

.. code-block:: python3

	from libmercury import Validator 
	class SigninValidator:
		pass

We can add fields to our validator using this syntax:

.. code-block:: python3

	from libmercury import Validator 
	class SigninValidator:
		username = Validator.String(min=3, max=20)
		password = Validator.String(min=8, max=50)

Using this syntax we have created 2 fields, username and password. We then added minimum and maximum character
limits to our fields. We can then integrate these fields into our controllers. So lets create a new controller named
AuthFlowController:

.. code-block:: bash

	python3 -m libmercury create controller AuthFlow

Then lets write the following code in our controller:

.. code-block:: python3

	from libmercury import GETRoute, POSTRoute, Request, Response, useValidator
	from src.validators.SigninValidator import SigninValidator
	class AuthFlowController:
		@staticmethod
		@useValidator(SigninValidator)
		@POSTRoute("/api/signin")
		def signin(request: Request) -> Response:
			response = Response("<h1>You passed the validator</h1>")
			return response

Lets then check if our validator is working by sending a POST request to our server:


.. code-block:: bash

	curl -X POST -d "username=ahsan" -d "password=password123" http://localhost:8000/api/signin


If everything was done correctly, we would see the following response:

.. code-block:: bash

	<h1>You passed the validator</h1>

Let's also create another one just for signing up:

.. code-block:: bash

	python3 -m libmercury create validator Signup

.. code-block:: python3

	from libmercury import Validator 
	class SignupValidator:
		username = Validator.String(min=3, max=20)
		password = Validator.String(min=8, max=50)

Next tutorial: `Tutorial 4 <tutorial4.html>`_
