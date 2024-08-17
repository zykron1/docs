Tutorial 2
==========
This tutorial will be based of the `previous tutorial <tutorial1.html>`_. Please complete it first.

In the last tutorial we leanred how to create a route and how to link it with our templates. In this tutorial we will
learn how to use the database in mercury.

Creating a Model
----------------

To create a model in Mercury, you can use the CLI:

.. code-block:: bash

	python3 -m libmercury create model user

This command generates a model called `user`. You can find the generated code in `src/cargo/user.py`:

.. code-block:: python3

	from libmercury.db import Column, Integer, Base

	class user(Base):
		__tablename__ = "user"
		id = Column(Integer, primary_key=True)

Adding Columns to Your Model
----------------------------

Let's extend the `user` model by adding `username` and `password` columns:

.. code-block:: python3

	from libmercury.db import Column, Integer, String, Base

	class user(Base):
		__tablename__ = "user"
		id = Column(Integer, primary_key=True)
		username = Column(String(20))
		password = Column(String(50))

.. warning::

	Never store passwords in plaintext. Use hashing instead. Mercury has built-in tools for this.

Creating a Migration
--------------------

After creating the model, you need to sync the database schema with the current state of your models. To do this, create a migration:

.. code-block:: bash

	python3 -m libmercury create migration first_migration

This command generates a migration file in `src/cargo/migrations/1.py`. Let's take a look at its contents:

.. code-block:: python3

	from libmercury.db import MigrationWrapper 

	_version = '1'
	_prev_version = None
	
	def upgrade(url):
		wrapper = MigrationWrapper(url)
	
	def downgrade(url):
		wrapper = MigrationWrapper(url)

The `upgrade` function is where you define the changes to apply to the database schema. The `downgrade` function defines how to revert those changes.

Using the MigrationWrapper
--------------------------

Let's start by creating the `user` table:

.. code-block:: python3

	from libmercury.db import MigrationWrapper, Column, Integer, String 

	_version = '1'
	_prev_version = None
	
	def upgrade(url):
		wrapper = MigrationWrapper(url)
		wrapper.create_table("user", [
			Column("id", Integer, primary_key=True),
			Column("username", String(20)),
			Column("password", String(50))
		])
	
	def downgrade(url):
		wrapper = MigrationWrapper(url)
		wrapper.delete_table("user")

Running the Migration
---------------------

To apply the migration, use the CLI:

.. code-block:: bash

	python3 -m libmercury migrate

If successful, you'll see output similar to:

.. code-block:: bash

	[Migrator] Running migration src/cargo/migrations/1.py
	[Migrator] Table 'user' created successfully.
	[Migrator] 'src/cargo/migrations/1.py' passed with no errors

And just like that, we can now use the user model in our code. To test this out, open a python shell in the root
project folder and try some of these commands:

.. code-block:: python3

	Python 3.10.6 (v3.10.6:9c7b4bd164, Aug	1 2022, 17:13:48) [Clang 13.0.0 (clang-1300.0.29.30)] on darwin
	Type "help", "copyright", "credits" or "license" for more information.
	>>> from src.cargo.connection import Connection
	>>> from src.cargo.userModel import user
	>>> new_user = user(username="JohnDoe", password="12345678")
	>>> Connection.Session.add_all([new_user,])
	>>> Connection.Session.commit()
	>>> Connection.Session.query(user).filter(user.username=="JohnDoe").first() # Using sqlalchemy syntax
	<src.cargo.userModel.user object at 0x10c28f730>
	>>> from libmercury import find_or_404
	>>> find_or_404(user, username="JohnDoe") # Using find_or_404
	<src.cargo.userModel.user object at 0x10c28f730>
	>>> find_or_404(user, username="nonexistant") # Using find_or_404
	<Response 22 bytes [404 NOT FOUND]>
	>>> from libmercury import exists
	>>> exists(user, username="JohnDoe") # Using exists
	True
	>>> exists(user, username="nonexistant") 
	False

Next tutorial: `Tutorial 3 <tutorial3.html>`_
