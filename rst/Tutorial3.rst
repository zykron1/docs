Tutorial 3
==========
This tutorial builds upon the `previous tutorial <tutorial2.html>`_. Please ensure you have completed it before proceeding.

In the previous tutorial, we covered the built-in ORM capabilities of Mercury. In this tutorial, we will explore validators. In Mercury, a validator is a preprocessor that can be attached to a controller to automatically filter out invalid requests. For instance, if you have a controller that accepts a username and a password, a validator can ensure that both fields are non-empty and that the input is a string. This preprocessing can be easily achieved using the CLI:

.. code-block:: bash

	python3 -m libmercury create validator Signin

After running this command, a new file will be created in `src/validators/SigninValidator.py`. Open the file, and you will see the following:

.. code-block:: python3

	from libmercury import Validator 
	class SigninValidator:
		pass

We can define fields for our validator using the following syntax:

.. code-block:: python3

	from libmercury import Validator 
	class SigninValidator:
		username = Validator.String(min=3, max=20)
		password = Validator.String(min=8, max=50)

In this example, we have created two fields, `username` and `password`, and set minimum and maximum character limits for each. These fields can then be integrated into our controllers. Let's create a new controller called `AuthFlowController`:

.. code-block:: bash

	python3 -m libmercury create controller AuthFlow

Next, add the following code to the controller:

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

Now, let's verify that our validator is working by sending a POST request to our server:

.. code-block:: bash

	curl -X POST -d "username=ahsan" -d "password=password123" http://localhost:8000/api/signin

If everything is set up correctly, you should see the following response:

.. code-block:: bash

	<h1>You passed the validator</h1>

Let's create another validator for signing up:

.. code-block:: bash

	python3 -m libmercury create validator Signup

Then, define the validator as follows:

.. code-block:: python3

	from libmercury import Validator 
	class SignupValidator:
		username = Validator.String(min=3, max=20)
		password = Validator.String(min=8, max=50)

Next tutorial: `Tutorial 4 <tutorial4.html>`_
